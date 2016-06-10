# Physical Pages

## struct page

* Every physical page of memory in linux is represented by a
  [struct page][page]:

```c
/*
 * Each physical page in the system has a struct page associated with
 * it to keep track of whatever it is we are using the page for at the
 * moment. Note that we have no way to track which tasks are using
 * a page, though if it is a pagecache page, rmap structures can tell us
 * who is mapping it.
 *
 * The objects in struct page are organized in double word blocks in
 * order to allows us to use atomic double word operations on portions
 * of struct page. That is currently only used by slub but the arrangement
 * allows the use of atomic double word operations on the flags/mapping
 * and lru list pointers also.
 */
struct page {
        /* First double word block */
        unsigned long flags;            /* Atomic flags, some possibly
                                         * updated asynchronously */
        union {
                struct address_space *mapping;  /* If low bit clear, points to
                                                 * inode address_space, or NULL.
                                                 * If page mapped as anonymous
                                                 * memory, low bit is set, and
                                                 * it points to anon_vma object:
                                                 * see PAGE_MAPPING_ANON below.
                                                 */
                void *s_mem;                    /* slab first object */
                atomic_t compound_mapcount;     /* first tail page */
                /* page_deferred_list().next     -- second tail page */
        };

        /* Second double word */
        struct {
                union {
                        pgoff_t index;          /* Our offset within mapping. */
                        void *freelist;         /* sl[aou]b first free object */
                        /* page_deferred_list().prev    -- second tail page */
                };

                union {
#if defined(CONFIG_HAVE_CMPXCHG_DOUBLE) && \
        defined(CONFIG_HAVE_ALIGNED_STRUCT_PAGE)
                        /* Used for cmpxchg_double in slub */
                        unsigned long counters;
#else
                        /*
                         * Keep _count separate from slub cmpxchg_double data.
                         * As the rest of the double word is protected by
                         * slab_lock but _count is not.
                         */
                        unsigned counters;
#endif

                        struct {

                                union {
                                        /*
                                         * Count of ptes mapped in mms, to show
                                         * when page is mapped & limit reverse
                                         * map searches.
                                         */
                                        atomic_t _mapcount;

                                        struct { /* SLUB */
                                                unsigned inuse:16;
                                                unsigned objects:15;
                                                unsigned frozen:1;
                                        };
                                        int units;      /* SLOB */
                                };
                                atomic_t _count;                /* Usage count, see below. */
                        };
                        unsigned int active;    /* SLAB */
                };
        };

        /*
         * Third double word block
         *
         * WARNING: bit 0 of the first word encode PageTail(). That means
         * the rest users of the storage space MUST NOT use the bit to
         * avoid collision and false-positive PageTail().
         */
        union {
                struct list_head lru;   /* Pageout list, eg. active_list
                                         * protected by zone->lru_lock !
                                         * Can be used as a generic list
                                         * by the page owner.
                                         */
                struct dev_pagemap *pgmap; /* ZONE_DEVICE pages are never on an
                                            * lru or handled by a slab
                                            * allocator, this points to the
                                            * hosting device page map.
                                            */
                struct {                /* slub per cpu partial pages */
                        struct page *next;      /* Next partial slab */
#ifdef CONFIG_64BIT
                        int pages;      /* Nr of partial slabs left */
                        int pobjects;   /* Approximate # of objects */
#else
                        short int pages;
                        short int pobjects;
#endif
                };

                struct rcu_head rcu_head;       /* Used by SLAB
                                                 * when destroying via RCU
                                                 */
                /* Tail pages of compound page */
                struct {
                        unsigned long compound_head; /* If bit zero is set */

                        /* First tail page only */
#ifdef CONFIG_64BIT
                        /*
                         * On 64 bit system we have enough space in struct page
                         * to encode compound_dtor and compound_order with
                         * unsigned int. It can help compiler generate better or
                         * smaller code on some archtectures.
                         */
                        unsigned int compound_dtor;
                        unsigned int compound_order;
#else
                        unsigned short int compound_dtor;
                        unsigned short int compound_order;
#endif
                };

#if defined(CONFIG_TRANSPARENT_HUGEPAGE) && USE_SPLIT_PMD_PTLOCKS
                struct {
                        unsigned long __pad;    /* do not overlay pmd_huge_pte
                                                 * with compound_head to avoid
                                                 * possible bit 0 collision.
                                                 */
                        pgtable_t pmd_huge_pte; /* protected by page->ptl */
                };
#endif
        };

        /* Remainder is not double word aligned */
        union {
                unsigned long private;          /* Mapping-private opaque data:
                                                 * usually used for buffer_heads
                                                 * if PagePrivate set; used for
                                                 * swp_entry_t if PageSwapCache;
                                                 * indicates order in the buddy
                                                 * system if PG_buddy is set.
                                                 */
#if USE_SPLIT_PTE_PTLOCKS
#if ALLOC_SPLIT_PTLOCKS
                spinlock_t *ptl;
#else
                spinlock_t ptl;
#endif
#endif
                struct kmem_cache *slab_cache;  /* SL[AU]B: Pointer to slab */
        };

#ifdef CONFIG_MEMCG
        struct mem_cgroup *mem_cgroup;
#endif

        /*
         * On machines where all RAM is mapped into kernel address space,
         * we can simply calculate the virtual address. On machines with
         * highmem some memory is mapped into kernel virtual memory
         * dynamically, so we need a place to store that address.
         * Note that this field could be 16 bits on x86 ... ;)
         *
         * Architectures with slow multiplication can define
         * WANT_PAGE_VIRTUAL in asm/page.h
         */
#if defined(WANT_PAGE_VIRTUAL)
        void *virtual;                  /* Kernel virtual address (NULL if
                                           not kmapped, ie. highmem) */
#endif /* WANT_PAGE_VIRTUAL */

#ifdef CONFIG_KMEMCHECK
        /*
         * kmemcheck wants to track the status of each byte in a page; this
         * is a pointer to such a status block. NULL if not tracked.
         */
        void *shadow;
#endif

#ifdef LAST_CPUPID_NOT_IN_PAGE_FLAGS
        int _last_cpupid;
#endif
}
/*
 * The struct page can be forced to be double word aligned so that atomic ops
 * on double words work. The SLUB allocator can make use of such a feature.
 */
#ifdef CONFIG_HAVE_ALIGNED_STRUCT_PAGE
        __aligned(2 * sizeof(unsigned long))
#endif
;
```

[page]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm_types.h#L44
