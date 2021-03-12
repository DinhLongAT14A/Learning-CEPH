# 1. Pools
- Pools are logical partitions for storing objects.
- When you first deploy a cluster without creating a pool, Ceph uses the default pools for storing data. A pool provides you with:
   + Resilience ( Khả năng nhân rộng ): You can set how many OSD are allowed to fail without losing data. For replicated ( Nhân rộng ) pools, it is the desired (mong muốn) number of copies/replicas ( bản sao ) of an object. A typical ( Điển hỉnh ) configuration stores an object and one additional (bổ sung ) copy (i.e., size = 2), but you can determine ( mục đích) the number of copies/replicas. For erasure (xóa !?) coded pools, it is the number of coding chunks (miếng) (i.e. m=2 in the erasure code profile)
   + Placement (vị trí) Groups: You can set the number of placement groups for the pool. A typical configuration uses approximately (Xâp xỉ) 100 placement groups per OSD to provide optimal (tối ưu ) balancing without using up too many computing resources. When setting up multiple ( bội số) pools, be careful to ensure (đảm bảo) you set a reasonable (hợp lý) number of placement groups for both the pool and the cluster as a whole.
   + CRUSH Rules: When you store data in a pool, placement of the object and its replicas (bản sao)(or chunks (mảnh) for erasure coded (mã hóa) pools) in your cluster is governed (điều hành) by CRUSH rules. You can create a custom CRUSH rule for your pool if the default rule is not appropriate (chấp nhận) for your use case.
   + Snapshots: When you create snapshots with ceph osd pool mksnap, you effectively (hiệu quả) take a snapshot of a particular (cụ thể) pool.
- To organize data into pools, you can list, create, and remove pools. You can also view the utilization (sự sử dụng) statistics (số liệu thống kê) for each pool.


# 2. List Pools
   
   To list your cluster's pools, execute: "ceph osd lspools"

# 3. Create a Pool
- Before creating pools, refer (tham khảo) to the Pool, PG and CRUSH Config Reference ( Tham chiếu cấu hình) .Ideally (Lý tưởng) , you should override (ghi đè) the default value for the number of placement groups in your Ceph configuration file, as the default is NOT ideal. For details ( Để biết chi tiết ) on placement group numbers refer to (tham khảo tới) setting the number of placement groups
*** note 
Starting with Luminous, all pools need to be associated (liên kết) to the application (ứng dụng) using the pool. See Associate Pool (Nhóm liên kết) to Application below (dưới đây) for more information.
For example:
    + osd pool default pg num = 100  ( pg !? )
    + osd pool default pgp num = 100 ( pgp !?)
- To create a pool, execute (thực hành):
  - ceph osd pool create {pool-name} [{pg-num} [{pgp-num}]] [replicated] \
     [crush-rule-name] [expected-num-objects]
  - ceph osd pool create {pool-name} [{pg-num} [{pgp-num}]]   erasure \
     [erasure-code-profile] [crush-rule-name] [expected_num_objects] [--autoscale-mode=<on,off,warn>]
     
Where:

   ``{pool-name}``     
   
   |Description ( Sự miêu tả) |The name of the pool. It must be unique (duy nhất)|
   |--------------------------|--------------------------------------------------|
   |Type                      |String                                            |                              
   |Required ( Cần thiết)     |Yes                                               |
      
``{pg-num}``

   |Description  |Tổng số nhóm của pool. Xem Placement Groups để biết chi tiết về cách tính một con số phù hợp. Giá trị mặc định 8 KHÔNG phù hợp với hầu hết các hệ thống. |
   |-------------|---------------------------------------------------------------------------------------------------------------------------------------------------------|
   |Type         |String                                                                                                                                                   |        |Required     |Yes                                                                                                                                                      |
   |Defualt      | 8                                                                                                                                                       |

``{pgp-num}``

``{replicated|erasure}``

``crush-rule-name]``

``[erasure-code-profile=profile]``

``--autoscale-mode=<on,off,warn>``


If you set the autoscale mode to ``on`` or ``warn``, you can let the system autotune or recommend changes to the number of placement groups in your pool based on actual usage.  If you leave it off, then you should refer to `Placement Groups`_ for more information.

``[expected-num-objects]``



# 4. Associate Pool to Application


Pools need to be associated with an application before use. Pools that will be
used with CephFS or pools that are automatically created by RGW are
automatically associated. Pools that are intended for use with RBD should be
initialized using the ``rbd`` tool (see `Block Device Commands`_ for more
information).

For other cases, you can manually associate a free-form application name to
a pool.::

        ceph osd pool application enable {pool-name} {application-name}

Note:: CephFS uses the application name ``cephfs``, RBD uses the
   application name ``rbd``, and RGW uses the application name ``rgw``.

# 5. Set Pool Quotas


You can set pool quotas for the maximum number of bytes and/or the maximum
number of objects per pool. ::

	ceph osd pool set-quota {pool-name} [max_objects {obj-count}] [max_bytes {bytes}]

For example::

	ceph osd pool set-quota data max_objects 10000

To remove a quota, set its value to ``0``.


# 6. Delete a Pool


To delete a pool, execute:

	ceph osd pool delete {pool-name} [{pool-name} --yes-i-really-really-mean-it]


To remove a pool the mon_allow_pool_delete flag must be set to true in the Monitor's
configuration. Otherwise they will refuse to remove a pool.

See `Monitor Configuration`_ for more information.



If you created your own rules for a pool you created, you should consider
removing them when you no longer need your pool::

	ceph osd pool get {pool-name} crush_rule

If the rule was "123", for example, you can check the other pools like so::

	ceph osd dump | grep "^pool" | grep "crush_rule 123"

If no other pools use that custom rule, then it's safe to delete that
rule from the cluster.

If you created users with permissions strictly for a pool that no longer
exists, you should consider deleting those users too::

	ceph auth ls | grep -C 5 {pool-name}
	ceph auth del {user}


# 7. Rename a Pool


To rename a pool, execute::

	ceph osd pool rename {current-pool-name} {new-pool-name}

If you rename a pool and you have per-pool capabilities for an authenticated
user, you must update the user's capabilities (i.e., caps) with the new pool
name.

# 8. Show Pool Statistics


To show a pool's utilization statistics, execute::

	rados df

Additionally, to obtain I/O information for a specific pool or all, execute::

        ceph osd pool stats [{pool-name}]


# 9. Make a Snapshot of a Pool


To make a snapshot of a pool, execute::

	ceph osd pool mksnap {pool-name} {snap-name}

# 10. Remove a Snapshot of a Pool


To remove a snapshot of a pool, execute::

	ceph osd pool rmsnap {pool-name} {snap-name}




# 11. Set Pool Values


To set a value to a pool, execute the following::

	ceph osd pool set {pool-name} {key} {value}

You may set values for the following keys:

``compression_algorithm``

``compression_mode``

``compression_min_blob_size``

``compression_max_blob_size``

``size``

``min_size``

``pg_num``

``pgp_num``

``crush_rule``

``allow_ec_overwrites``

``hashpspool``

``nodelete``

``nopgchange``

``nosizechange``

``write_fadvise_dontneed``

``noscrub``

``nodeep-scrub``

``hit_set_type``

``hit_set_count``

``hit_set_period``

``hit_set_fpp``

``cache_target_dirty_ratio``

``cache_target_dirty_high_ratio``

``cache_target_full_ratio``

``target_max_bytes``

``target_max_objects``

``hit_set_grade_decay_rate``

``hit_set_search_last_n``

``cache_min_flush_age``

``cache_min_evict_age``

``fast_read``

``scrub_min_interval``

``scrub_max_interval``

``deep_scrub_interval``

``recovery_priority``

``recovery_op_priority``

# 12. Get Pool Values

To get a value from a pool, execute the following::

	ceph osd pool get {pool-name} {key}

You may get values for the following keys:

``size``

``min_size``

``pg_num``

``pgp_num``

``crush_rule``

``hit_set_type``

``hit_set_count``

``hit_set_period``

``hit_set_fpp``

``cache_target_dirty_ratio``

``cache_target_dirty_high_ratio``

``cache_target_full_ratio``

``target_max_bytes``

``target_max_objects``

``cache_min_flush_age``

``cache_min_evict_age``

``fast_read``

``scrub_min_interval``

``scrub_max_interval``

``deep_scrub_interval``

``allow_ec_overwrites``

``recovery_priority``

``recovery_op_priority``

# 13. Set the Number of Object Replicas
 

To set the number of object replicas on a replicated pool, execute the following::

	ceph osd pool set {poolname} size {num-replicas}

Note : important : The ``{num-replicas}`` includes the object itself.
   If you want the object and two copies of the object for a total of
   three instances of the object, specify ``3``.

For example::

	ceph osd pool set data size 3

You may execute this command for each pool. **Note:** An object might accept
I/Os in degraded mode with fewer than ``pool size`` replicas.  To set a minimum
number of required replicas for I/O, you should use the ``min_size`` setting.
For example:

  ceph osd pool set data min_size 2

This ensures that no object in the data pool will receive I/O with fewer than
``min_size`` replicas.


# 14. Get the Number of Object Replicas
 
To get the number of object replicas, execute the following::

	ceph osd dump | grep 'replicated size'

Ceph will list the pools, with the ``replicated size`` attribute highlighted.
By default, ceph creates two replicas of an object (a total of three copies, or
a size of 3).





