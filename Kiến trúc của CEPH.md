# 1.CEPH RADOS
 RADOS (Reliable Autonomic Distributed Object Store) là trung tâm của CEPH storage system, cũng được gọi là CEPH Storage Cluster.
 
 RADOS cung cấp rất nhiều tính năng quan trọng cho Ceph, bao gồm phân bố lưu trữ đối tượng, HA, bảo đảm, chịu lỗi, tự xử lý, tự giám sát. Vì vậy, RADOS layer cực kỳ quan trọng trong kiến trúc Ceph storage. Tất cả các phương thức truy cập vd: RBD, CephFS, RADOSGW, librados đều hoạt động trên RADOS layer.
 
 Khi Ceph Cluster nhận yêu cầu ghi từ client, thuật toán CRUSH sẽ tính toán vị trị data sẽ được lưu. Thông tin này sau đó sẽ được gửi tới RADOS layer cho việc xử lý sau. Dựa trên tập rule CRUSH, RADOS phân bố data trên tất cả cluster node và trong những đối tượng nhỏ. Cuối cùng obj được lưu trên OSDs.
 
 RADOS sẽ chịu trách nhiệm giữ data bảo đảm khi cấu hình và nhân bản factor nhiều hơn 1 lần. Tại cùng thời điểm, nó sẽ nhân bản object, tạo bảo sao, lưu trữ tại phân vùng khác. Tuy nhiên, để tùy chỉnh và có mức bảo đảm cao hơn, ta cần tuy chỉnh CRUSH ruleset theo yêu cầu và hạ tầng. RADOS bảo đảm sẽ luôn có nhiều hơn 1 bản sao lưu trên RADOS cluster.
 
 Bên cạnh lưu và nhân bản object tới các cluster, RADOS sẽ bảo đảm trạng thái obj phù hợp. Trong trường hợp sai khác, sẽ khôi phục obj dựa trên các bản sao. Tính năng khôi phục chạy tự động vì Ceph có 2 cơ chế tự quản trị, tự sửa lỗi.
 
 Khi nhìn vào kiến trúc Ceph, ta sẽ thầy nó gồm 2 phần, RADOS là tầng dưới, nằm trong Ceph cluster, không giao triếp trực tiếp với client, bên trên sẽ có client interface.
 
 Rados gồm 2 thành phần nhỏ chính là Ceph OSD và Ceph MON
 
  # 1.1. CEPH OSD - CEPH Object Storage Device
   # a. Khái niệm 
   - Một trong những thành phần quan trọng khi xây dựng blocks trong Ceph storage cluster. Nó lưu trữ data thực sự trên physical disk drives tại mỗi cluster node dạng obj. Phần lớn hoạt động bên trong Ceph Cluster được thực hiện bởi tiến trình Ceph OSD.
   - Ceph OSD lưu tất cả client data dạng obj, đáp ứng yêu cầu yêu cầu đến data được lưu trữ. Ceph cluster bao gồm nhiều OSD. Trên mỗi hoạt động đọc và ghi, client request tới cluster maps từ monitors, sau đó, họ sẽ tương tác với OSDs với hoạt động đọc ghi, không có sự can thiệp monitors. Điều này kiến tiến trình xử lý dữ liệu nhanh hơn khi ghi tới OSD, lưu trữ data trực tiếp mà không thông qua các lớp xử lý data khác. Cơ chế data-storage-and-retrieval mechanism gần như độc nhất khi so sánh ceph với các công cụ khác.
   # b. Tính năng của CEPH
   - Tính năng cơ sở của Ceph bao gồm reliability, rebalancing, recovery, consistency tới từ các OSD. Dựa trêm cấu hình replication size, Ceph cung cấp sự bảo đảm bằng cách nhân bản mỗi obj tới các cluster node, khiến obj có tính HA và có thể chịu lỗi. Mỗi Obj trong OSD đều có phiên bản chính và các bản sao nằm trên các OSD khác. Vì Ceph lưu trữ phân tán nên các obj được lưu trữ trên nhiều OSD, mỗi OSD sẽ chứa 1 số phiên bản chính của Obj và bản phụ 1 số obj khác.
   - Khi sảy ra lỗi disk, Ceph OSD daemon sẽ so sánh các OSD để bắt đầu hoạt động khôi phục. Trong thời điểm, OSD lưu trữ bản sao sẽ trở thành bản chính, và tạo bản sao mới tới OSD khác trong thời điểm khôi phục. Cơ bản Ceph cluster sẽ tạo 1 OSD daemon cho mỗi disk trong cluster. Tuy nhiên OSD hỗ trợ tùy chỉnh, cho phép 1 OSD per host, disk, raid volume. Hầu hết khi triển khai Ceph trong JBOD environment sẽ sử dụng mỗi OSD daemon trên mỗi disk vật lý.
   # c. Cấu trúc của CEPH OSD
   - Ceph OSD bao gồm Ceph OSD filesystem, Linux filesystem nằm phía trên và Ceph OSD service. Linux filesystem góp phần quan trọng tới Ceph OSD daemon như hỗ trợ extended attributes (XATTRs). filesystems' extended attributes cung cấp các thông tin nội bộ về obj state, snap shot, metadata, ACL tới OSD daemon, cho phép quản trị data
   
     ![ceph-arch-2](https://user-images.githubusercontent.com/75653012/110295520-c9a24e00-8023-11eb-94f7-8557c03147cf.png)
     
   - Hoạt động Ceph OSD bên trên physical disk drive có phân vùng trong Linux partition. Linux partition có thể là Btrfs (B-tree file system), XFS, or ext4. Việc lựa chọn filesystem góp phần lớn trong việc tính toán hiệu năng trên Ceph Cluster, mỗi file system đều có đặc điểm riêng.
     + Btrfs: OSD với Btrfs filesystem, cung cấp hiệu năng tốt nhất khi so sánh với XFS, ext4. Điểm mạnh khi sử dụng Btrfs là nó hỗ trợ copy-on-write, writable snapshots hỗ trợ các VM provisioning and cloning. Đồng thời, nó hỗ trợ transparent compression (nén ..), pervasive checksums (kiểm tra), và incorporates multidevice management (quản lý ..) trong fs. Btrfs hỗ trợ hiệu quả XATTRs, inline data cho nhưng file nhỏ, provides integrated volume management như SSD aware, and has the demanding feature of online fsck. Mặc dù có những tính năng mới này nhưng Btrfs vẫn chưa sẵn sàng trong production nhưng có thể dùng trong test deployment.
     + XFS: Tin cậy, rõ ràng, fs ổn định vững chắc. Được khuyên dùng cho hệ thống XFS Ceph Cluster và được sử dụng nhiều nhất trên Ceph storage và khuyên dùng cho mỗi OSDs. Tuy nhiên nó vẫn còn 1 số đặc điểm kém Btfs và một số vần đề về hiệu suất khi mở rộng metadata. XFS là journaling filesystem, vì thế, mỗi khi client gửi data to write tới Ceph cluster, đầu tiên nó sẽ ghi vào journaling space sau đó mới là XFS file system. Điều này làm tăng chi phi về data cung như kiến XFS chạy chậm hơn Btrfs.
     + Ext4: fourth extended filesystem, hỗ trợ journaling filesystem, hỗ trợ tốt Ceph OSD; Tuy nhiên nó không thân thiện bằng XFS. Từ góc độ hiệu năng, ext4 chưa bằng Btrfs.
  - Ceph OSD sử dụng các thuộc tính mở rộng của FS để biết trạng thái obj nội bộ và metadata. XATTRs cho phép lưu thông tin mở rộng liên quan tới object dạng xattr_name và xattr_value, cung cấp tagging objects with more metadata information. The ext4 filesystem không cung cấp đủ tính năng XATTRs dẫn đến 1 số hạn chế về số lượng bytes lưu trữ XATTRs, vì thế khiến ext4 không thân thiện khi lựa chọn FS. Đồng thời Btrfs and XFS hỗ trợ khả năng lưu trữ lớn hơn so với XATTRs. 
   
   # d. Ceph OSD journal
   Ceph sử dụng journaling filesystems như Btrfs, XFS cho OSD. Trước khi đẩy data tới backing store, Ceph ghi data tới 1 phân vùng đặc biệt gọi journal. Nó là small buffer-sized partition cách biệt với spinning disk as OSD hoặc trên SSD disk or partition hoặc là 1 file trên fs. Trong kỹ thuật, Ceph ghi tất cả tới jounal, sau đó mới lưu trừ tới backing storage.
   
   ![ceph-arch-3](https://user-images.githubusercontent.com/75653012/110296200-a330e280-8024-11eb-98ef-84eb4262a764.png)
   
   10gb là size cơ bản của journal, có thể to hơn tùy vào partition. Ceph sử dụng journals để tăng tốc độ và tăng tính bảo đảm. Journal cho phép Ceph OSD thực hiện công việc lưu trư nhanh hơn; random write được ghi trên sequential pattern on journals, sau đó đẩy sang FS. Điều này kiến filesystem có đủ thời gian để kết hợp ghi xuống disk. Hiệu suất được cải thiện khi journal được thiết lập trên SSD disk partition. Theo kịch bản, tất cả client sẽ được ghi rất nhanh trên SSD journal sau đó đẩy xuống đĩa quay.

   Sử dụng SSD như journals cho phép OSD xử lý được 1 khối lượng công việc lớn. Tùy nhiên nếu journals chậm hơn backing store, nó sẽ hạn chế hiệu năng của cluster. Vì thế theo yêu cầu, không vượt quá tỷ lệ 4-5 OSDs trên mỗi journal disk khi sử dụng SSDs mở rộng cho journals. Vượt quá OSD trên mỗi journal disk có thể tạo hiện tượng nghẽn cổ trai cho cluster.

   Trong trường hợp lỗi journal trong Btrfs-based filesystem, nó sẽ giảm thiểu mất mát dữ liệu. Btrfs sử dụng kỹ thuật copy-on-write filesystem, vì thế nếu nội dung content block thay đổi, việc ghi sẽ diễn ra riêng biệt. Trong trường hợp journal gặp lỗi, data sẽ vẫn tồn tại.

   Không sử dụng RAID cho ceph vì:
     + Chạy RAID và nhân bản trên raid làm giảm hiệu năng cũng như ổ cứng khiến việc nhân bản diễn ra gắp 2 lần và chiếm nhiều tài nguyên. Vì thế nếu cấu hình RAID, khuyên dụng sử dụng RAID 0.
     + Bảo vệ data, Ceph sẽ chịu trách nhiệm nhân bản, tái tạo data thay vì RAID, Ceph vượt trội so với RAID truyền thống, nhanh chóng khôi phục giảm tốn kém phần cứng
Hiệu năng Ceph sẽ giảm xuống khi sử dụng RAID 5 6 vì tính chất random IO

