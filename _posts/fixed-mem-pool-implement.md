title: 定长内存池的实现
date: 2016-03-30 07:43:08
tags:
---
# 原由
由于游戏后台有大量复杂的逻辑以及数据，有些涉及到充值才能得到的道具以及游戏代币
等信息，而这些数据是不能够丢失的，但游戏服务器又经常发布，不可能保证100%不出bug,
而某些bug可能导致服务器进程终止，带来数据丢失，引发严重问题。
为了解决这个问题，避免数据丢失，游戏服务器通常5分钟回写数据库作持久化，5分钟内的
数据存储在共享内存中，这样即使进程意外退出，重启进程还是可以恢复数据。但这也带来
了一个新的问题,内存分配要自己来完全掌控，如何高效组织与分配共享内存中的内存呢，
这就是本篇文章的目的。

# 区别
共享内存考虑到恢复，而通常恢复之后的内存地址不一定与之前的数据内存地址一样，通常
来说，就是不一样。所以在共享内存中不能用简单的指针（存储的是绝对地址）来构建数据
结构。
## 共享内存的创建
首先共享内存的创建主要有2个函数,头文件`<sys/shm.h>`
- `shmget(key_t key, size_t size, int shmflg);`
> *说明*：创建一个共享内存并返回该共享内存的标识符

    * key为整型值,用户指定，恢复也是依赖这个key
    * size为申请共享内存的大小
    * shmflg为标志位，如：IPC_CREAT | IPC_EXCL | 0644 表示key的共享内存如果存在
就报错(去掉IPC_EXCL则不报错，依然创建)，设置errno为EEXIST，如果不存在就创建
 
- `shmat(int shmid, const void *shmaddr, int shmflg);`
> *说明*: 连接标识符为shmid的共享内存，并将共享内存对象映射到调用进程的地址空间
随后可像本地空间一样访问, 返回共享内存的起始地址
    * shmid为shmget返回的标识符
    * shmaddr指定共享内存的内存地址，通常设为NULL让系统指定
    * shmflg设置读写权限

由于shmat每次将共享内存映射到进程地址空间的虚拟地址都不一样，所以，这决定了在共享
内存不能使用普通的指针（绝对地址）来设计内存池的结构。

# 实现
*虽然shmat每次映射的共享内存的起始地址不一样，但在共享内部的相对地址并没有改变*,
基于这一特点，在共享内存内部可以使用偏移量pos来记录在共享内存的相对地址。基于这
样的要点来设计共享内存，思路就清晰了。
## 内存池结构
内存池为头部和数据部分，数据部分由多个定长节点构成，定义如下：
```cpp
/* 内存池头信息 */
    typedef struct MemPoolHeader
    {
    	/* 最大可分配大小 */
    	size_t max_size;
        /* 最大节点数 */
        uint32_t max_node;
        /* 节点实际大小 */
        size_t node_size;
        /* 块大小 */
        size_t block_size;
        /* 已分配块数 */
        size_t alloc_block;
        /* 已分配空间大小 */
        size_t alloc_size;
        /* 空闲块链 */
        size_t free_list;
    } Header;

    /* 分配块结构 */
    typedef struct AllocBlock
    {
        /* 偏移量 */
        size_t next;
        /* 存储用户自定义数据部分 */
        char node[0];       
    } Block;
```
> 注：AllocBlock中的node为用户自定义数据的起始位置，AllocBlock的实际大小会在Init
时根据offsetof(AllocBlock, node) + node_size按照8B对其然后存在Header的block_size
里，也就是AllocBlock块的实际大小，定长也指的是这个定长。

再说一下在整个实现中极其重要的2个方法:
```cpp
inline size_t Ref(void* p) const
{
	return (size_t)((intptr_t)p -(intptr_t)(header_));
}

inline void* Deref(size_t pos) const
{
	return ((char*)header_ + pos);
}
```
- Ref: 由指针位置p计算相对于header_的偏移量
- Deref: 由相对位置计算实际内存地址
这里贴出最主要的几个方法的实现吧
```cpp
// 初始化
int FixedMemPool::Init(void* mem, uint32_t max_node, size_t node_size, bool recovery)
{
    return_fail_if_null(mem, -1);
    header_ = (Header*)mem;
    if (recovery)
    {
        // 内存池头信息验证错误
        if (!CheckHeaderValid(header_, max_node, node_size))
        {
            std::cout << "recover MemPool failed" << std::endl;
            return ERR_RECOVERY;
        }
    }
    else
    {
        // 初始化内存池头
    	header_->max_size = GetAllocSize(max_node, node_size);
        header_->max_node = max_node;
        header_->node_size = node_size;
        header_->block_size = GetBlockSize(node_size);
        header_->alloc_block = 0;
        header_->alloc_size = sizeof(Header);
        header_->free_list = INVALID_PTR;
    }
    data_ = (char*)header_ + sizeof(Header);

    return 0;
}

// 分配内存块
void* FixedMemPool::Alloc(bool zero)
{
    Block *p = GetFreeBlock();
    // 内存池中分配块
    if (!p)
    {
    	// 内存池已满
    	if (header_->alloc_block + 1 > header_->max_node)
    	{
    		std::cout << "No enough memory for new alloc" << std::endl;
    		return NULL;
    	}
    	// 内存池中分配块
    	p = (Block*)Deref(header_->alloc_size);
    	header_->alloc_block += 1;
    	header_->alloc_size += header_->block_size;
    }
    if (!IsValidBlock(p))
    {
    	std::cout << "Alloc error, Block alloc failed" << std::endl;
    	return NULL;
    }

    if (zero)
    {
        memset(p, 0, header_->block_size);
    }

    p->next = 0;

    return p->node;
}

// 回收内存块
int FixedMemPool::Free(void* node)
{
    Block* block = (Block*)((intptr_t) node - offsetof(Block, node));
    if (IsUsedBlock(block))
    {
    	block->next = header_->free_list;
    	header_->free_list = Ref(block);
    }
    else
    {
    	std::cout << "free node error，node is not valid or used" << std::endl;
    	return -1;
    }
    return 0;
}
```
源码见[github地址](https://github.com/viclab/fixed_mem_pool)
