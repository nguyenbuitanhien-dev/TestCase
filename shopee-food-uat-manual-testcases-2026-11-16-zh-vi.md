# ShopeeFood 对接 UAT 手工测试用例<br>Bộ ca kiểm thử thủ công tích hợp UAT ShopeeFood

> 环境 / Môi trường: scp https://scp.a3jom9ul.top | 门店 / Cửa hàng: M000001 | POS 登录账号须有「S 外卖平台配送」权限 / Tài khoản đăng nhập POS phải có quyền “Giao hàng trên nền tảng giao đồ ăn S”

## 一、商品管理（菜单同步）<br>I. Quản lý món (Đồng bộ thực đơn)

| 编号<br>Mã | 用例<br>Ca kiểm thử | 操作步骤<br>Các bước thực hiện | 预期结果<br>Kết quả mong đợi | 结果<br>Kết quả |
|---|---|---|---|---|
| A01 | 页面加载<br>Tải trang | 进入 ShopeeFood 商品管理页（POS：设置→ShopeeFood 商品管理；Web：菜品管理→ShopeeFood）<br>Truy cập trang Quản lý món ShopeeFood (POS: Cài đặt→Quản lý món ShopeeFood; Web: Quản lý món→ShopeeFood) | 页面正常打开；显示环境=UAT、连接状态=已连接<br>Trang mở bình thường; hiển thị Môi trường=UAT, Trạng thái kết nối=Đã kết nối | |
| A02 | 概览状态计数<br>Số lượng theo trạng thái ở phần tổng quan | 查看概览状态行<br>Xem hàng trạng thái tổng quan | 已同步/待同步/待核对/待审核/部分成功/失败/未映射 7 项计数与列表实际状态一致<br>Số lượng của 7 trạng thái Đã đồng bộ/Chờ đồng bộ/Cần đối chiếu/Chờ duyệt/Thành công một phần/Thất bại/Chưa ánh xạ khớp với trạng thái thực tế trong danh sách | |
| A03 | 最近完整同步卡<br>Thẻ lần đồng bộ hoàn chỉnh gần nhất | 查看「最近完整同步」行，点击任务号<br>Xem hàng “Đồng bộ hoàn chỉnh gần nhất”, nhấp vào mã tác vụ | 显示末次同步时间与成功/失败计数；点击弹出结果弹窗<br>Hiển thị thời gian đồng bộ gần nhất và số lượng thành công/thất bại; nhấp vào sẽ mở cửa sổ kết quả | |
| A04 | 查看问题跳转<br>Chuyển đến mục xem vấn đề | 点击「查看问题 →」<br>Nhấp “Xem vấn đề →” | 跳转异常与补偿页菜单同步 Tab，可见失败任务及明细<br>Chuyển đến tab Đồng bộ thực đơn trên trang Ngoại lệ và bù trừ; có thể xem tác vụ thất bại và chi tiết | |
| A05 | 从菜品库添加<br>Thêm từ thư viện món | 点「从菜品库添加」，勾选本地菜品，确认<br>Nhấp “Thêm từ thư viện món”, chọn món tại cửa hàng rồi xác nhận | 菜品进入渠道菜单，菜品 Tab 列表出现该菜，状态=待同步/未映射<br>Món được thêm vào thực đơn kênh và xuất hiện trong danh sách tab Món; trạng thái=Chờ đồng bộ/Chưa ánh xạ | |
| A06 | 无 SPU 编码的菜<br>Món không có mã SPU | 渠道菜不填 SPU 编码直接同步<br>Đồng bộ trực tiếp món trên kênh mà không nhập mã SPU | 该菜同步失败；补齐编码后重试成功<br>Món đó đồng bộ thất bại; sau khi bổ sung mã và thử lại thì thành công | |
| A07 | 发起完整同步<br>Khởi chạy đồng bộ hoàn chỉnh | 点「完整同步」<br>Nhấp “Đồng bộ hoàn chỉnh” | 按钮置灰显示执行中，顶部状态条显示进度；完成后自动弹出结果弹窗<br>Nút chuyển sang màu xám và hiển thị Đang thực hiện; thanh trạng thái phía trên hiển thị tiến độ; khi hoàn tất, cửa sổ kết quả tự động mở | |
| A08 | 同步结果计数<br>Số lượng trong kết quả đồng bộ | 查看结果弹窗<br>Xem cửa sổ kết quả | 显示已同步/已删除/失败 3 项计数；有失败时显示部分成功<br>Hiển thị số lượng của 3 mục Đã đồng bộ/Đã xóa/Thất bại; nếu có mục thất bại thì hiển thị Thành công một phần | |
| A09 | 失败原因查看<br>Xem nguyên nhân thất bại | 结果弹窗或菜品 Tab 点「查看原因」<br>Nhấp “Xem nguyên nhân” trong cửa sổ kết quả hoặc tab Món | 弹窗显示失败原因原文<br>Cửa sổ hiển thị nguyên văn lý do thất bại | |
| A10 | 重新执行完整同步<br>Thực hiện lại đồng bộ hoàn chỉnh | 结果弹窗点「重新执行完整同步」<br>Nhấp “Thực hiện lại đồng bộ hoàn chỉnh” trong cửa sổ kết quả | 仅重新提交失败项，生成新任务<br>Chỉ gửi lại các mục thất bại và tạo tác vụ mới | |
| A11 | 全量覆盖<br>Ghi đè toàn bộ | 同步后登录 SPF 商家后台核对<br>Đăng nhập vào cổng quản lý người bán SPF để kiểm tra sau khi đồng bộ | 我方未推送的存量菜被删除；菜单分类/名称/价格/图片与我方一致<br>Các món hiện có không được hệ thống chúng tôi đẩy lên sẽ bị xóa; danh mục/tên/giá/hình ảnh của thực đơn khớp với hệ thống của chúng tôi | |
| A12 | 营业时间核对<br>Kiểm tra giờ hoạt động | SPF 商家后台查看营业时间<br>Xem giờ hoạt động trên cổng quản lý người bán SPF | 与我方门店营业时间一致（含多时段）<br>Khớp với giờ hoạt động của cửa hàng chúng tôi (bao gồm nhiều khung giờ) | |
| A13 | 执行中防重复<br>Ngăn thao tác trùng khi đang chạy | 同步执行中再次点击「完整同步」<br>Nhấp lại “Đồng bộ hoàn chỉnh” khi quá trình đồng bộ đang chạy | 按钮置灰不可重复发起<br>Nút chuyển sang màu xám và không thể khởi chạy lại | |
| A14 | 菜品列表列完整<br>Danh sách món có đủ các cột | 查看菜品 Tab 列表<br>Xem danh sách trong tab Món | 图片/名称/分类/价格/同步状态/审核状态/partnerId/ShopeeId 列齐全<br>Có đủ các cột Hình ảnh/Tên/Danh mục/Giá/Trạng thái đồng bộ/Trạng thái duyệt/partnerId/ShopeeId | |
| A15 | 同步状态显示<br>Hiển thị trạng thái đồng bộ | 逐行核对同步状态<br>Kiểm tra trạng thái đồng bộ trên từng hàng | 已同步绿/待同步/失败红/未映射灰/分类未映射，与 SPF 后台实际一致<br>Đã đồng bộ màu xanh lá/Chờ đồng bộ/Thất bại màu đỏ/Chưa ánh xạ màu xám/Danh mục chưa ánh xạ; khớp với trạng thái thực tế trên cổng SPF | |
| A16 | 审核状态刷新<br>Làm mới trạng thái duyệt | 审核列点「刷新」<br>Nhấp “Làm mới” tại cột Duyệt | 审核状态实时更新；已拒绝行显示「查看原因」<br>Trạng thái duyệt được cập nhật theo thời gian thực; các hàng bị từ chối hiển thị “Xem nguyên nhân” | |
| A17 | 单体重试<br>Thử lại từng món | 失败行→查看原因→点「重试」<br>Hàng thất bại→Xem nguyên nhân→Nhấp “Thử lại” | 仅该菜重新提交；成功后弹窗关闭、列表状态刷新<br>Chỉ món đó được gửi lại; khi thành công, cửa sổ đóng và trạng thái trong danh sách được làm mới | |
| A18 | 批量重试失败<br>Thử lại hàng loạt các mục thất bại | 勾选多行（含失败与非失败）→「批量重试失败」<br>Chọn nhiều hàng (gồm cả thất bại và không thất bại)→“Thử lại hàng loạt các mục thất bại” | 仅失败项重提；弹出结果汇总；勾选无非失败项时提示<br>Chỉ gửi lại các mục thất bại; hiển thị tổng hợp kết quả; nếu chỉ chọn các mục không thất bại thì có thông báo | |
| A19 | 取消选择<br>Bỏ chọn | 勾选后点「取消选择」<br>Chọn các hàng rồi nhấp “Bỏ chọn” | 勾选清空，已选计数归零<br>Xóa toàn bộ lựa chọn và số lượng mục đã chọn trở về 0 | |
| A20 | 搜索筛选<br>Tìm kiếm và lọc | 按菜品名称/编码搜索；按同步状态筛选<br>Tìm kiếm theo tên/mã món; lọc theo trạng thái đồng bộ | 结果正确<br>Kết quả chính xác | |
| A21 | 下架菜不推送<br>Món đã ngừng bán không được đẩy lên | 将渠道菜下架后执行完整同步<br>Ngừng bán món trên kênh rồi thực hiện đồng bộ hoàn chỉnh | 该菜不进推送，SPF 侧对应下架/删除<br>Món đó không được đẩy lên; món tương ứng trên SPF bị ngừng bán/xóa | |
| A22 | 加料 Tab<br>Tab Món thêm | 查看加料列表<br>Xem danh sách món thêm | 名称/所属组/价格/Shopee 状态/partnerId/ShopeeId/同步状态/审核列齐全<br>Có đủ các cột Tên/Nhóm trực thuộc/Giá/Trạng thái Shopee/partnerId/ShopeeId/Trạng thái đồng bộ/Duyệt | |
| A23 | 加料售罄<br>Món thêm hết hàng | POS 将加料设售罄 2 小时后恢复<br>Trên POS, đặt món thêm thành hết hàng, sau 2 giờ khôi phục | SPF 侧加料对应售罄/恢复<br>Món thêm tương ứng trên SPF chuyển sang hết hàng/được khôi phục | |
| A24 | 加料组/分组 Tab<br>Tab Nhóm món thêm/Nhóm món | 查看加料组、菜品分组 Tab<br>Xem tab Nhóm món thêm và Nhóm món | 仅展示被在售菜品引用的分组<br>Chỉ hiển thị các nhóm được các món đang bán tham chiếu | |
| A25 | 加料编辑<br>Chỉnh sửa món thêm | 加料行点编辑，修改保存<br>Nhấp Chỉnh sửa trên hàng món thêm, sửa rồi lưu | 保存成功，列表刷新<br>Lưu thành công và danh sách được làm mới | |

## 二、订单接入<br>II. Tiếp nhận đơn hàng

| 编号<br>Mã | 用例<br>Ca kiểm thử | 操作步骤<br>Các bước thực hiện | 预期结果<br>Kết quả mong đợi | 结果<br>Kết quả |
|---|---|---|---|---|
| B01 | 新单接入<br>Tiếp nhận đơn mới | 在 SPF UAT 门店下单<br>Đặt hàng tại cửa hàng UAT trên SPF | POS 订单管理→Shopee Tab 出现该单（待接单）；语音播报「有新的订单」；打印机出票<br>Đơn xuất hiện trong POS Quản lý đơn hàng→tab Shopee (Chờ nhận đơn); có thông báo thoại “Có đơn hàng mới”; máy in in phiếu | |
| B02 | 订单详情<br>Chi tiết đơn hàng | 打开订单详情<br>Mở chi tiết đơn hàng | 显示菜品明细（名称/数量/价格）、金额、客户信息、第三方单号<br>Hiển thị chi tiết món (tên/số lượng/giá), số tiền, thông tin khách hàng và mã đơn của bên thứ ba | |
| B03 | 收银页明细<br>Chi tiết trên trang thanh toán | 进入该单收银页<br>Truy cập trang thanh toán của đơn hàng | 商品明细正常显示<br>Chi tiết món hiển thị bình thường | |
| B04 | 重复推送幂等<br>Tính lũy đẳng khi đẩy lại | 同一订单推送重发<br>Gửi lại thông báo đẩy cho cùng một đơn hàng | 不重复建单<br>Không tạo trùng đơn hàng | |
| B05 | 金额字段核对<br>Kiểm tra các trường số tiền | 核对订单金额明细<br>Đối chiếu chi tiết số tiền của đơn hàng | 菜品金额/佣金/实收等与 SPF 后台一致<br>Tiền món/hoa hồng/số tiền thực nhận, v.v. khớp với cổng SPF | |
| B06 | 未映射菜品的单<br>Đơn có món chưa ánh xạ | 用 SPF 侧未映射菜品下单<br>Đặt hàng bằng món chưa ánh xạ trên SPF | 我方不落库；POS 消息中心收到告警，可跳转菜品 Tab<br>Hệ thống chúng tôi không lưu đơn; Trung tâm tin nhắn POS nhận cảnh báo và có thể chuyển đến tab Món | |

## 三、订单操作<br>III. Thao tác đơn hàng

| 编号<br>Mã | 用例<br>Ca kiểm thử | 操作步骤<br>Các bước thực hiện | 预期结果<br>Kết quả mong đợi | 结果<br>Kết quả |
|---|---|---|---|---|
| C01 | 接单<br>Nhận đơn | POS 对待接订单点接单<br>Trên POS, nhấp Nhận đơn đối với đơn đang chờ nhận | SPF 后台订单变已接单/备餐中；本地订单状态推进<br>Đơn trên cổng SPF chuyển sang Đã nhận/Đang chuẩn bị; trạng thái đơn hàng cục bộ được cập nhật | |
| C02 | 拒单-缺货<br>Từ chối đơn - Hết hàng | 拒单选「缺货」并勾选缺货菜品<br>Chọn từ chối với lý do “Hết hàng” và chọn các món hết hàng | SPF 侧订单取消；勾选菜品被置售罄<br>Đơn trên SPF bị hủy; các món được chọn chuyển sang trạng thái hết hàng | |
| C03 | 拒单-缺货未选菜<br>Từ chối đơn - Hết hàng nhưng không chọn món | 拒单选「缺货」但不勾选菜品<br>Chọn từ chối với lý do “Hết hàng” nhưng không chọn món | 拦截提示，不允许提交<br>Hiển thị thông báo chặn và không cho phép gửi | |
| C04 | 拒单-太忙/关店<br>Từ chối đơn - Quá bận/Đóng cửa | 拒单选「太忙」或「关店」<br>Chọn từ chối với lý do “Quá bận” hoặc “Đóng cửa” | SPF 侧订单取消；门店状态对应联动<br>Đơn trên SPF bị hủy; trạng thái cửa hàng được cập nhật tương ứng | |
| C05 | 出餐<br>Hoàn tất món | 订单备餐完成点出餐<br>Khi chuẩn bị món xong, nhấp Đã sẵn sàng/Đã hoàn tất món | SPF 侧状态更新；已被骑手取走的单提示「已被骑手取走」不报错<br>Trạng thái trên SPF được cập nhật; đơn đã được tài xế lấy sẽ hiển thị “Đã được tài xế lấy” mà không báo lỗi | |
| C06 | SPF 侧取消<br>Hủy từ phía SPF | 在 SPF 后台取消订单<br>Hủy đơn hàng trên cổng SPF | 我方订单同步取消<br>Đơn hàng của chúng tôi được đồng bộ sang trạng thái đã hủy | |
| C07 | 超时接单<br>Nhận đơn quá hạn | 超过确认时限后再接单<br>Nhận đơn sau khi đã quá thời hạn xác nhận | 可读提示超时，本地订单已取消<br>Hiển thị thông báo dễ hiểu về việc quá hạn; đơn hàng cục bộ đã bị hủy | |

## 四、营业状态<br>IV. Trạng thái hoạt động

| 编号<br>Mã | 用例<br>Ca kiểm thử | 操作步骤<br>Các bước thực hiện | 预期结果<br>Kết quả mong đợi | 结果<br>Kết quả |
|---|---|---|---|---|
| D01 | 暂停接单<br>Tạm dừng nhận đơn | 渠道页设置暂停营业，截止时间≤次日 05:00<br>Trên trang kênh, đặt tạm dừng hoạt động với thời gian kết thúc≤05:00 ngày hôm sau | SPF 后台门店变暂停；页面显示「暂停至」时间<br>Cửa hàng trên cổng SPF chuyển sang Tạm dừng; trang hiển thị thời gian “Tạm dừng đến” | |
| D02 | 暂停时间非法<br>Thời gian tạm dừng không hợp lệ | 截止时间填超过次日 05:00<br>Nhập thời gian kết thúc muộn hơn 05:00 ngày hôm sau | 输入即拦截提示，不提交<br>Hiển thị thông báo chặn ngay khi nhập và không gửi dữ liệu | |
| D03 | 恢复接单<br>Khôi phục nhận đơn | 渠道页点恢复<br>Nhấp Khôi phục trên trang kênh | SPF 后台门店恢复可下单<br>Cửa hàng trên cổng SPF khôi phục khả năng nhận đơn | |

## 五、异常与补偿 / 审计<br>V. Ngoại lệ và bù trừ / Kiểm toán

| 编号<br>Mã | 用例<br>Ca kiểm thử | 操作步骤<br>Các bước thực hiện | 预期结果<br>Kết quả mong đợi | 结果<br>Kết quả |
|---|---|---|---|---|
| E01 | 异常与补偿页<br>Trang Ngoại lệ và bù trừ | 打开异常与补偿页，切 5 个 Tab<br>Mở trang Ngoại lệ và bù trừ, chuyển qua 5 tab | 事件/菜单同步/补偿/审计/概览数据正常显示<br>Dữ liệu Sự kiện/Đồng bộ thực đơn/Bù trừ/Kiểm toán/Tổng quan hiển thị bình thường | |
| E02 | 失败事件重导入<br>Nhập lại sự kiện thất bại | 对处理失败的 webhook 事件点重导入<br>Nhấp Nhập lại đối với sự kiện webhook xử lý thất bại | 重新处理；成功后状态变成功<br>Xử lý lại; khi thành công, trạng thái chuyển sang Thành công | |
| E03 | 审计脱敏<br>Ẩn dữ liệu nhạy cảm trong kiểm toán | 审计 Tab 打开任意调用详情<br>Trong tab Kiểm toán, mở chi tiết của một lệnh gọi bất kỳ | token/密钥等敏感字段显示为 ***<br>Các trường nhạy cảm như token/khóa hiển thị là *** | |
| E04 | 漏单补偿<br>Bù trừ đơn bị bỏ sót | SPF 有单但我方无单时等待补偿扫描<br>Khi SPF có đơn nhưng hệ thống chúng tôi không có đơn, chờ quét bù trừ | 补偿任务自动补录；重复单跳过<br>Tác vụ bù trừ tự động ghi bổ sung đơn; đơn trùng được bỏ qua | |
