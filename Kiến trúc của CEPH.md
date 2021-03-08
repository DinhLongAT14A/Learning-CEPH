# 1. Ceph RADOS
- RADOS (Reliable Autonomic Distributed Object Store) là trung tâm của Ceph storage system, cũng được gọi là Ceph Storage Cluster.
- RADOS cung cấp rất nhiều tính năng quan trọng cho Ceph, bao gồm phân bố lưu trữ đối tượng, HA, bảo đảm, chịu lỗi, tự xử lý, tự giám sát. Vì vậy, RADOS layer cực kỳ quan trọng trong kiến trúc Ceph storage. Tất cả các phương thức truy cập vd: RBD, CephFS, RADOSGW, librados đều hoạt động trên RADOS layer.
- Khi Ceph Cluster nhận yêu cầu ghi từ client, thuật toán CRUSH sẽ tính toán vị trị data sẽ được lưu. Thông tin này sau đó sẽ được gửi tới RADOS layer cho việc xử lý sau. Dựa trên tập rule CRUSH, RADOS phân bố data trên tất cả cluster node và trong những đối tượng nhỏ. Cuối cùng obj được lưu trên OSDs.
- RADOS sẽ chịu trách nhiệm giữ data bảo đảm khi cấu hình và nhân bản factor nhiều hơn 1 lần. Tại cùng thời điểm, nó sẽ nhân bản object, tạo bảo sao, lưu trữ tại phân vùng khác. Tuy nhiên, để tùy chỉnh và có mức bảo đảm cao hơn, ta cần tuy chỉnh CRUSH ruleset theo yêu cầu và hạ tầng. RADOS bảo đảm sẽ luôn có nhiều hơn 1 bản sao lưu trên RADOS cluster.
- Bên cạnh lưu và nhân bản object tới các cluster, RADOS sẽ bảo đảm trạng thái obj phù hợp. Trong trường hợp sai khác, sẽ khôi phục obj dựa trên các bản sao. Tính năng khôi phục chạy tự động vì Ceph có 2 cơ chế tự quản trị, tự sửa lỗi.
- Khi nhìn vào kiến trúc Ceph, ta sẽ thầy nó gồm 2 phần, RADOS là tầng dưới, nằm trong Ceph cluster, không giao triếp trực tiếp với client, bên trên sẽ có client interface.
  
Rados gồm 2 thành phần nhỏ chính là Ceph OSD và Ceph MON

