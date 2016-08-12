---
layout: post
title: The journey of a ZFS read() call in 1 min [μ]
date: 2016-08-11 12:00:00 -0600
year: 2016
month: 8
day: 11
published: true
tags: zfs dtrace illumos micropost
summary: Printing ZFS's read() code path with DTrace
---

### Motivation & Objective

I've been reading up a lot on ZFS recently. As a part
of this study, I'm planning to watch a video where Matt
Ahrens goes through the
[read and write codepaths on ZFS](https://www.youtube.com/watch?v=ptY6-K78McY).
Before watching it though, I thought it would be cool if I went
through the code by myself. Then, hopefully, any questions
that I have while reading the code will be addressed in the
video code review.

As I am becoming more familiar with illumos and its internals
, I decided to use DTrace. My main objective was to create a
map of the codepath so I can have a better sense of direction
while reading the code.

### The D script

I quickly hacked a script together to achieve my aforementioned
objective. One thing that I had to consider was whether I should
also include the `plockstat` provider and include all the locking
points on my map. I decided not to do that. Even though I'm
planning to understand the reason behind using the different locks
on the critical points that I will encounter, my objective
is not to read the implementation of these mechanisms. Furthermore,
their output on my map would probably be distracting.

The easiest way that I could think of issuing a clean read()
system call was by using `cat` on a file. Therefore, my DTrace
script tries to detect only the first read() call issued by
a `cat` command, so its output is hopefully reproducable.

{% highlight perl %}
syscall::read:entry
/execname == "cat"/
{
	self->t = 1;
}

syscall::read:return
/self->t/
{
	self->t = 0;
	exit(0);
}

sdt:::, fbt:::entry, fbt:::return
/self->t/
{
}
{% endhighlight %}

### Result

If you are curious what the output looks like check
the Appendix section. illumos kernel devs won't find
anything surprising or out of the ordinary here. The
main point here is that in very little time I was
able to create a map of the path taken by a system
call. I'm just barely scratching the surface here
in the number of aids that DTrace can provide me
while I am trying to understand a system that is
being developed for more than a decade. Off to read
some code now!

### Appendix: Output

The following output was produced in my OmniOS
VM running on vagrant provided by OmniTI. I also
used the `-F` option of the dtrace utility for a
better visualization of the path.

{% highlight perl %}
CPU FUNCTION                                 
  0  -> read32                                
  0    -> read                                
  0      -> getf                              
  0        -> set_active_fd                   
  0        <- set_active_fd                   
  0        -> set_active_fd                   
  0        <- set_active_fd                   
  0      <- getf                              
  0      -> nbl_need_check                    
  0      <- nbl_need_check                    
  0      -> fop_rwlock                        
  0        -> fs_rwlock                       
  0        <- fs_rwlock                       
  0      <- fop_rwlock                        
  0      -> fop_read                          
  0        -> crgetmapped                     
  0        <- crgetmapped                     
  0        -> zfs_read                        
  0          -> rrm_enter_read                
  0            -> rrw_enter_read              
  0              -> rrw_enter_read_impl       
  0              <- rrw_enter_read_impl       
  0            <- rrw_enter_read              
  0          <- rrm_enter_read                
  0          -> zfs_range_lock                
  0            -> kmem_alloc                  
  0              -> kmem_cache_alloc          
  0              <- kmem_cache_alloc          
  0            <- kmem_alloc                  
  0            -> avl_numnodes                
  0            <- avl_numnodes                
  0            -> avl_add                     
  0              -> avl_find                  
  0              <- avl_find                  
  0              -> avl_insert                
  0              <- avl_insert                
  0            <- avl_add                     
  0          <- zfs_range_lock                
  0          -> vn_has_cached_data            
  0          <- vn_has_cached_data            
  0          -> sa_get_db                     
  0          <- sa_get_db                     
  0          -> dmu_read_uio_dbuf             
  0            -> zrl_add_impl                
  0            <- zrl_add_impl                
  0            -> dmu_read_uio_dnode          
  0              -> dmu_buf_hold_array_by_dnode 
  0                -> kmem_zalloc             
  0                  -> kmem_cache_alloc      
  0                  <- kmem_cache_alloc      
  0                <- kmem_zalloc             
  0                -> zio_root                
  0                  -> zio_null              
  0                    -> zio_create          
  0                      -> kmem_cache_alloc  
  0                      <- kmem_cache_alloc  
  0                      -> mutex_init        
  0                      <- mutex_init        
  0                      -> cv_init           
  0                      <- cv_init           
  0                      -> list_create       
  0                      <- list_create       
  0                      -> list_create       
  0                      <- list_create       
  0                      -> zfs_zone_zio_init 
  0                      <- zfs_zone_zio_init 
  0                    <- zio_create          
  0                  <- zio_null              
  0                <- zio_root                
  0                -> dbuf_whichblock         
  0                <- dbuf_whichblock         
  0                -> dbuf_hold               
  0                  -> dbuf_hold_level       
  0                    -> dbuf_hold_impl      
  0                      -> dbuf_find         
  0                        -> dbuf_hash       
  0                        <- dbuf_hash       
  0                      <- dbuf_find         
  0                      -> arc_buf_add_ref   
  0                        -> buf_hash        
  0                        <- buf_hash        
  0                        -> add_reference   
  0                          -> arc_buf_type  
  0                          <- arc_buf_type  
  0                          -> multilist_remove 
  0                            -> arc_state_multilist_index_func 
  0                              -> buf_hash  
  0                              <- buf_hash  
  0                              -> multilist_get_num_sublists 
  0                              <- multilist_get_num_sublists 
  0                            <- arc_state_multilist_index_func 
  0                           | multilist_remove:multilist-remove 
  0                           | multilist_remove:multilist-remove 
  0                            -> mutex_owned 
  0                            <- mutex_owned 
  0                            -> multilist_sublist_remove 
  0                              -> list_remove 
  0                              <- list_remove 
  0                            <- multilist_sublist_remove 
  0                          <- multilist_remove 
  0                        <- add_reference   
  0                       | arc_buf_add_ref:arc-hit 
  0                       | arc_buf_add_ref:arc-hit 
  0                        -> arc_access      
  0                          -> ddi_get_lbolt 
  0                            -> lbolt_event_driven 
  0                              -> gethrtime 
  0                                -> tsc_gethrtime 
  0                                <- tsc_gethrtime 
  0                              <- gethrtime 
  0                            <- lbolt_event_driven 
  0                          <- ddi_get_lbolt 
  0                        <- arc_access      
  0                      <- arc_buf_add_ref   
  0                    <- dbuf_hold_impl      
  0                  <- dbuf_hold_level       
  0                <- dbuf_hold               
  0                -> dbuf_read               
  0                  -> zrl_add_impl          
  0                  <- zrl_add_impl          
  0                  -> zrl_remove            
  0                  <- zrl_remove            
  0                <- dbuf_read               
  0                -> dmu_zfetch              
  0                <- dmu_zfetch              
  0                -> zio_wait                
  0                  -> zio_execute           
  0                    -> zio_ready           
  0                      -> zio_wait_for_children 
  0                      <- zio_wait_for_children 
  0                      -> zio_wait_for_children 
  0                      <- zio_wait_for_children 
  0                      -> zio_walk_parents  
  0                        -> list_head       
  0                        <- list_head       
  0                      <- zio_walk_parents  
  0                    <- zio_ready           
  0                    -> zio_done            
  0                      -> zio_wait_for_children 
  0                      <- zio_wait_for_children 
  0                      -> zio_wait_for_children 
  0                      <- zio_wait_for_children 
  0                      -> zio_wait_for_children 
  0                      <- zio_wait_for_children 
  0                      -> zio_wait_for_children 
  0                      <- zio_wait_for_children 
  0                      -> zio_inherit_child_errors 
  0                      <- zio_inherit_child_errors 
  0                      -> zio_inherit_child_errors 
  0                      <- zio_inherit_child_errors 
  0                      -> zio_inherit_child_errors 
  0                      <- zio_inherit_child_errors 
  0                      -> zio_pop_transforms 
  0                      <- zio_pop_transforms 
  0                      -> vdev_stat_update  
  0                      <- vdev_stat_update  
  0                      -> zio_inherit_child_errors 
  0                      <- zio_inherit_child_errors 
  0                      -> zio_gang_tree_free 
  0                      <- zio_gang_tree_free 
  0                      -> zio_walk_parents  
  0                        -> list_head       
  0                        <- list_head       
  0                      <- zio_walk_parents  
  0                      -> cv_broadcast      
  0                      <- cv_broadcast      
  0                    <- zio_done            
  0                  <- zio_execute           
  0                  -> zio_destroy           
  0                    -> list_destroy        
  0                    <- list_destroy        
  0                    -> list_destroy        
  0                    <- list_destroy        
  0                    -> mutex_destroy       
  0                    <- mutex_destroy       
  0                    -> cv_destroy          
  0                    <- cv_destroy          
  0                    -> kmem_cache_free     
  0                    <- kmem_cache_free     
  0                  <- zio_destroy           
  0                <- zio_wait                
  0              <- dmu_buf_hold_array_by_dnode 
  0              -> uiomove                   
  0                -> xcopyout_nta            
  0                <- kcopy                   
  0              <- uiomove                   
  0              -> dmu_buf_rele_array        
  0                -> dbuf_rele               
  0                  -> dbuf_rele_and_unlock  
  0                    -> arc_buf_freeze      
  0                    <- arc_buf_freeze      
  0                    -> arc_released        
  0                    <- arc_released        
  0                    -> arc_buf_remove_ref  
  0                      -> buf_hash          
  0                      <- buf_hash          
  0                      -> remove_reference  
  0                        -> arc_buf_type    
  0                        <- arc_buf_type    
  0                        -> multilist_insert 
  0                          -> arc_state_multilist_index_func 
  0                            -> buf_hash    
  0                            <- buf_hash    
  0                            -> multilist_get_num_sublists 
  0                            <- multilist_get_num_sublists 
  0                          <- arc_state_multilist_index_func 
  0                         | multilist_insert:multilist-insert 
  0                         | multilist_insert:multilist-insert 
  0                          -> mutex_owned   
  0                          <- mutex_owned   
  0                          -> multilist_sublist_insert_head 
  0                            -> list_insert_head 
  0                            <- list_insert_head 
  0                          <- multilist_sublist_insert_head 
  0                        <- multilist_insert 
  0                      <- remove_reference  
  0                    <- arc_buf_remove_ref  
  0                    -> arc_buf_eviction_needed 
  0                    <- arc_buf_eviction_needed 
  0                  <- dbuf_rele_and_unlock  
  0                <- dbuf_rele               
  0                -> kmem_free               
  0                  -> kmem_cache_free       
  0                  <- kmem_cache_free       
  0                <- kmem_free               
  0              <- dmu_buf_rele_array        
  0            <- dmu_read_uio_dnode          
  0            -> zrl_remove                  
  0            <- zrl_remove                  
  0          <- dmu_read_uio_dbuf             
  0          -> zfs_range_unlock              
  0            -> zfs_range_unlock_reader     
  0              -> avl_remove                
  0              <- avl_remove                
  0              -> kmem_free                 
  0                -> kmem_cache_free         
  0                <- kmem_cache_free         
  0              <- kmem_free                 
  0            <- zfs_range_unlock_reader     
  0          <- zfs_range_unlock              
  0          -> zfs_tstamp_update_setup       
  0            -> gethrestime                 
  0              -> pc_gethrestime            
  0                -> gethrtime               
  0                  -> tsc_gethrtime         
  0                  <- tsc_gethrtime         
  0                <- gethrtime               
  0              <- pc_gethrestime            
  0            <- gethrestime                 
  0          <- zfs_tstamp_update_setup       
  0          -> rrm_exit                      
  0            -> rrw_exit                    
  0              -> cv_broadcast              
  0              <- cv_broadcast              
  0            <- rrw_exit                    
  0          <- rrm_exit                      
  0        <- zfs_read                        
  0      <- fop_read                          
  0      -> fop_rwunlock                      
  0        -> fs_rwunlock                     
  0        <- fs_rwunlock                     
  0      <- fop_rwunlock                      
  0      -> releasef                          
  0        -> clear_active_fd                 
  0        <- clear_active_fd                 
  0        -> cv_broadcast                    
  0        <- cv_broadcast                    
  0      <- releasef                          
  0    <- read
{% endhighlight %}

