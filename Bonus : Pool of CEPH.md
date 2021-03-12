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
      {pool-name}     
      
      |Description|The name of the pool. It must be unique|
      |Type       |String                                 |                              
      |Required   |Yes                                    |
      

