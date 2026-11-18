# C端扫码自主预订桌台 手工测试用例(中越双语)
# Test case kiểm thử thủ công: Khách tự đặt bàn qua mã QR (Song ngữ Trung - Việt)

> 涉及端 Phạm vi:Web-C = Zalo 内 H5(diancan_zalo)/ Web-C = H5 trong Zalo (diancan_zalo);Web-M = POS booking 模块 / Web-M = module booking trên POS;BE = vietnam-takeout-server。
> 前置条件 Điều kiện tiên quyết:商户已在订桌配置开启「允许客户自助订桌」(默认关)/ Merchant đã bật "Cho phép khách tự đặt bàn" trong cấu hình đặt bàn (mặc định tắt);客户身份为 Zalo 会员 / Khách là thành viên Zalo。
> 注意 Lưu ý:Zalo `phone_number` 权限为外部依赖,未开通时 B01/B02 用例改用测试环境已绑定手机号的会员验证「复用」分支 / Quyền `phone_number` của Zalo là phụ thuộc ngoài; nếu chưa được cấp, case B01/B02 dùng thành viên đã ràng buộc SĐT sẵn để kiểm tra nhánh "dùng lại".
> 依据 Căn cứ:docs/specs/customer-self-reservation.md(AC-1~AC-32)。

## 一、商家端:二维码与配置 / Web-M: Mã QR & cấu hình

| 编号 STT | 端 | 用例 Tên case | 操作步骤 Các bước | 预期结果 Kết quả mong đợi | 结果 KQ |
|---|---|---|---|---|---|
| Q01 | Web-M | 订桌二维码弹窗(M1)<br>Popup mã QR đặt bàn (M1) | 预订列表工具栏点「订桌二维码」<br>Bấm "Mã QR đặt bàn" trên thanh công cụ danh sách đặt bàn | 弹窗含说明文案、本商户二维码图、预约页链接;「复制链接」「下载图片」可用;二维码内容含 `mid` 参数、静态且商户内唯一;开关关闭时显示「开启后才有效」提示但二维码仍可见可下载<br>Popup có mô tả, ảnh QR của merchant, link trang đặt bàn; nút "Sao chép liên kết"/"Tải ảnh" hoạt động; QR chứa tham số `mid`, tĩnh và duy nhất mỗi merchant; khi đang tắt cấu hình vẫn hiện QR kèm ghi chú "chỉ có hiệu lực sau khi bật" | |
| Q02 | Web-M/BE | 无预订权限拦截<br>Chặn tài khoản không có quyền | 用无预订管理权限的账号访问二维码接口<br>Dùng tài khoản không có quyền quản lý đặt bàn gọi API mã QR | 接口返回 403,不产生二维码数据;前端入口不展示<br>API trả 403, không sinh dữ liệu QR; nút trên giao diện không hiển thị | |
| Q03 | Web-M | 订桌配置新增两行(M2)<br>2 dòng cấu hình mới (M2) | 打开「订桌配置」页查看并修改保存<br>Mở trang "Cấu hình đặt bàn", xem, sửa và lưu | 存在「允许客户自助订桌」开关(默认关)与「可提前预订天数(天)」(默认 7);与既有配置行同页保存,保存后 C 端按新配置生效<br>Có công tắc "Cho phép khách tự đặt bàn" (mặc định tắt) và "Số ngày được đặt trước" (mặc định 7); lưu cùng trang với các cấu hình cũ, sau lưu Web-C áp dụng cấu hình mới | |
| Q04 | Web-C/BE | 开关关闭全流程拦截<br>Chặn toàn luồng khi tắt cấu hình | 关闭开关后,客户扫码打开预约页并尝试调创建接口<br>Tắt công tắc, khách quét QR mở trang đặt bàn và thử gọi API tạo | C 端展示「本店暂未开放线上预订」(R8),无可选桌台与提交入口,仍可「查看菜单」;创建接口返回 `SELF_RESERVE_DISABLED`;商家端历史预订、统计、黑名单不受影响<br>Web-C hiện "Cửa hàng chưa mở đặt bàn trực tuyến" (R8), không có bàn/nút gửi, vẫn xem được menu; API tạo trả `SELF_RESERVE_DISABLED`; lịch sử đặt bàn/thống kê/danh sách đen trên Web-M không bị ảnh hưởng | |

## 二、C端预约页与选桌 / Web-C: Trang đặt bàn & chọn bàn

| 编号 STT | 端 | 用例 Tên case | 操作步骤 Các bước | 预期结果 Kết quả mong đợi | 结果 KQ |
|---|---|---|---|---|---|
| R01 | Web-C | 预约页 R1 展示<br>Hiển thị trang đặt bàn R1 | 开关开启状态下扫码进入 R1<br>Bật công tắc rồi quét QR vào trang R1 | 展示商户名称、地址、营业时间、可预订日期(chips 横滑)、午餐/晚餐时段起止、用餐时长说明;点「查看菜单」跳该商户 2C 菜单页,全程不要求登录、不要求绑定手机号<br>Hiện tên cửa hàng, địa chỉ, giờ mở cửa, dải ngày đặt được (chips trượt ngang), khung giờ bữa trưa/tối, thời lượng dùng bữa; bấm "Xem menu" mở trang menu 2C, không yêu cầu đăng nhập hay ràng buộc SĐT | |
| R02 | Web-C/BE | 可用桌台查询<br>Tra cứu bàn trống | 选择某日午餐时段进入选桌页 R2<br>Chọn ngày + khung giờ trưa, vào trang chọn bàn R2 | 仅列出该日期时段内空闲桌台;已锁定、时段冲突的桌台不出现;判定结果与商家端 `availableTables` 一致<br>Chỉ liệt kê bàn trống trong ngày/khung giờ đó; bàn bị khóa hoặc trùng khung giờ không hiện; kết quả khớp với `availableTables` phía Web-M | |
| R03 | Web-C/BE | 越界日期拦截<br>Chặn ngày ngoài phạm vi | 选择早于商户当天或超出可提前天数的日期<br>Chọn ngày trước hôm nay hoặc vượt quá số ngày đặt trước cho phép | 不给可选日期;接口返回空列表或业务错误,不产生占台<br>Không cho chọn ngày đó; API trả danh sách rỗng hoặc lỗi nghiệp vụ, không giữ bàn | |
| R04 | Web-C | 用餐人数录入<br>Nhập số khách | 在 R2 选桌页用 Stepper 录入人数<br>Nhập số khách bằng stepper ở trang chọn bàn R2 | 人数 ≥1;随所选桌台容量给出上限提示;该值随创建接口提交<br>Số khách ≥1; cảnh báo giới hạn theo sức chứa của bàn đã chọn; giá trị được gửi kèm khi tạo đặt bàn | |
| R05 | Web-C | 无空桌空态<br>Trạng thái trống khi hết bàn | 选择无空桌的日期时段<br>Chọn ngày/khung giờ không còn bàn trống | 展示空态页,引导改选其他时段或日期<br>Hiện trang trạng thái trống, gợi ý chọn khung giờ/ngày khác | |

## 三、手机号绑定与创建预订 / Ràng buộc SĐT & tạo đặt bàn

| 编号 STT | 端 | 用例 Tên case | 操作步骤 Các bước | 预期结果 Kết quả mong đợi | 结果 KQ |
|---|---|---|---|---|---|
| B01 | Web-C | 未绑定强制授权,拒绝后阻断<br>Chưa ràng buộc SĐT, từ chối ủy quyền bị chặn | 未绑定手机号的会员提交预订,在 Zalo 授权弹窗拒绝<br>Thành viên chưa ràng buộc SĐT gửi đặt bàn, từ chối trong popup ủy quyền Zalo | 阻断提交并提示需绑定手机号;数据库无新增预订记录、桌台占用不变;可放弃授权返回继续浏览菜单<br>Chặn gửi và thông báo cần ràng buộc SĐT; không có bản ghi đặt bàn mới, bàn không bị giữ; có thể bỏ ủy quyền và quay lại xem menu | |
| B02 | Web-C/BE | 已绑定复用<br>Dùng lại SĐT đã ràng buộc | 已完成绑定的会员直接提交预订<br>Thành viên đã ràng buộc gửi đặt bàn trực tiếp | 不再拉起授权;联系电话自动填充已绑定手机号,只读展示(锁定图标)不可手改;创建成功<br>Không hỏi ủy quyền lại; SĐT liên hệ tự điền số đã ràng buộc, chỉ đọc (icon khóa), không sửa được; tạo thành công | |
| B03 | Web-C/BE | 创建成功<br>Tạo đặt bàn thành công | 绑定手机号后,选空闲桌、人数 4、填姓名提交<br>Sau ràng buộc, chọn bàn trống, 4 khách, điền tên và gửi | 返回预订详情,状态=已确认,`source=customer`;商家端预订列表可见该记录<br>Trả về chi tiết, trạng thái = Đã xác nhận, `source=customer`; bản ghi hiện trong danh sách đặt bàn Web-M | |
| B04 | Web-C/BE | 非法输入拦截<br>Chặn dữ liệu không hợp lệ | 人数填 0 或联系人姓名为空提交;再绕过前端直接调接口<br>Gửi với số khách = 0 hoặc tên trống; sau đó gọi API trực tiếp bỏ qua frontend | 前端阻止提交;直接调接口后端返回 400,不产生记录<br>Frontend chặn gửi; gọi API trực tiếp trả 400, không tạo bản ghi | |
| B05 | BE | 当日次数上限<br>Giới hạn số lần trong ngày | 同一手机号当日预订达 `daily_max_reservations` 后再次提交<br>Cùng SĐT đạt `daily_max_reservations` lần trong ngày rồi gửi tiếp | 返回 `DAILY_LIMIT_REACHED`,不产生记录、不占用桌台;C 端 Toast 展示对应文案<br>Trả `DAILY_LIMIT_REACHED`, không tạo bản ghi, không giữ bàn; Web-C hiện Toast tương ứng | |
| B06 | BE | 黑名单拦截<br>Chặn danh sách đen | 用黑名单手机号提交预订<br>Gửi đặt bàn bằng SĐT trong danh sách đen | 返回 `BLACKLIST_REJECTED`,不创建记录<br>Trả `BLACKLIST_REJECTED`, không tạo bản ghi | |
| B07 | BE | 并发占台<br>Tranh chấp bàn đồng thời | 两个客户对同一桌台同一时段并发提交<br>Hai khách gửi đồng thời cho cùng bàn, cùng khung giờ | 仅一个成功,另一个返回 `TABLE_TIME_CONFLICT`<br>Chỉ một yêu cầu thành công, yêu cầu còn lại trả `TABLE_TIME_CONFLICT` | |
| B08 | BE | 锁定桌台拦截<br>Chặn bàn bị khóa | 对已锁定的桌台提交预订<br>Gửi đặt bàn cho bàn đang bị khóa | 返回 `TABLE_LOCKED`,不创建记录<br>Trả `TABLE_LOCKED`, không tạo bản ghi | |

## 四、我的预订与取消 / Đặt bàn của tôi & hủy

| 编号 STT | 端 | 用例 Tên case | 操作步骤 Các bước | 预期结果 Kết quả mong đợi | 结果 KQ |
|---|---|---|---|---|---|
| M01 | Web-C | 「我的订座」入口<br>Mục "Đặt bàn của tôi" | 打开 C 端账户页查看功能列表并点击入口<br>Mở trang tài khoản Web-C, xem danh sách chức năng và bấm vào mục | 账户页存在「我的订座」入口(位于积分商城与设置支付密码之间);点击进入我的预订列表 R5;无任何预订时展示空态页<br>Trang tài khoản có mục "Đặt bàn của tôi" (nằm giữa Sàn đổi điểm và Đặt mật khẩu thanh toán); bấm vào danh sách R5; không có đặt bàn nào thì hiện trang trạng thái trống | |
| M02 | Web-C | 列表与详情<br>Danh sách & chi tiết | 在 R5 查看本人名下各状态预订并打开详情 R6<br>Xem danh sách đặt bàn các trạng thái ở R5 và mở chi tiết R6 | 列表覆盖各状态(已确认/已到店/已完成/已取消/爽约);详情含商户、桌台、日期、时段、人数、状态、预选明细及「尚未加入购物车/已加入购物车」标记<br>Danh sách đủ các trạng thái (Đã xác nhận/Đã đến/Hoàn thành/Đã hủy/Không đến); chi tiết gồm cửa hàng, bàn, ngày, khung giờ, số khách, trạng thái, món chọn trước và nhãn "Chưa thêm vào giỏ/Đã thêm vào giỏ" | |
| M03 | Web-C/BE | 取消已确认预订<br>Hủy đặt bàn đã xác nhận | 对一笔「已确认」的本人预订执行取消(二次确认弹窗)<br>Hủy một đặt bàn "Đã xác nhận" của chính mình (popup xác nhận lần 2) | 状态变为「已取消」,桌台占用释放;商家端列表与操作日志同步可见<br>Trạng thái chuyển "Đã hủy", giải phóng bàn; danh sách và nhật ký thao tác trên Web-M đồng bộ | |
| M04 | Web-C/BE | 不可取消的状态<br>Trạng thái không được hủy | 对已到店/已完成/已取消/爽约的预订尝试取消<br>Thử hủy đặt bàn Đã đến/Hoàn thành/Đã hủy/Không đến | 已到店等状态无取消按钮;绕过前端调接口返回业务错误,状态与桌台占用不变<br>Các trạng thái này không có nút hủy; gọi API trực tiếp trả lỗi nghiệp vụ, trạng thái và bàn không đổi | |

## 五、分享 / Chia sẻ

| 编号 STT | 端 | 用例 Tên case | 操作步骤 Các bước | 预期结果 Kết quả mong đợi | 结果 KQ |
|---|---|---|---|---|---|
| S01 | Web-C/BE | 免登录查看+脱敏<br>Xem không cần đăng nhập + che thông tin | 在预订详情底部弹层生成/复制分享链接,用未登录、未绑定的第三人打开<br>Tạo/sao chép link chia sẻ ở chi tiết đặt bàn, mở bằng người thứ ba chưa đăng nhập | 可直接查看商户名称、桌台名、日期、时段、人数、状态及预选商品名称/数量/单价快照与预估合计;姓名脱敏仅首字符(如 阮*),手机号仅前 4 位(如 0912 *** ***);页面只读;提供「我也要订位」入口跳 R1<br>Xem được tên cửa hàng, tên bàn, ngày, khung giờ, số khách, trạng thái, món chọn trước (tên/số lượng/đơn giá snapshot) và tổng ước tính; tên chỉ hiện ký tự đầu, SĐT chỉ 4 số đầu; trang chỉ đọc; có nút "Tôi cũng muốn đặt bàn" về R1 | |
| S02 | Web-C/BE | 失效令牌<br>Token hết hiệu lực | 打开令牌不存在或记录已删除的分享链接<br>Mở link chia sẻ có token không tồn tại hoặc bản ghi đã xóa | 展示「预约不存在或已失效」页,HTTP 404,不暴露任何预订数据<br>Hiện trang "Đặt bàn không tồn tại hoặc đã hết hiệu lực", HTTP 404, không lộ dữ liệu | |
| S03 | Web-C/BE | 状态实时反映<br>Trạng thái cập nhật theo thời gian thực | 客户取消预订后,被分享人再次打开链接<br>Sau khi khách hủy, người được chia sẻ mở lại link | 页面显示最新状态「已取消」<br>Trang hiện trạng thái mới nhất "Đã hủy" | |

## 六、预约预选菜品 / Chọn món trước khi đặt bàn

| 编号 STT | 端 | 用例 Tên case | 操作步骤 Các bước | 预期结果 Kết quả mong đợi | 结果 KQ |
|---|---|---|---|---|---|
| P01 | Web-C/BE | 预选流程<br>Quy trình chọn món trước | R1 点「查看菜单」进预选页 P1,选 2 个商品(Stepper 数量),点「确认菜品」返回 R1 并提交预约<br>Từ R1 bấm "Xem menu" vào P1, chọn 2 món (stepper số lượng), bấm "Xác nhận món" quay lại R1 và gửi đặt bàn | R1 呈已选态「已选 N 道菜 · 预估合计 · 修改」;预约创建成功且详情返回 2 条预选明细;不生成订单、桌台购物车为空、商品库存不变<br>R1 hiện "Đã chọn N món · tổng ước tính · Sửa"; đặt bàn thành công, chi tiết có 2 dòng món chọn trước; không tạo đơn, giỏ bàn trống, tồn kho không đổi | |
| P02 | Web-C/BE | 逐菜备注<br>Ghi chú từng món | P1 点击菜品打开「菜品备注」页录入;P2「查看已预选菜品」页逐菜改数量与备注<br>Ở P1 bấm món mở trang "Ghi chú món" để nhập; ở P2 "Xem món đã chọn" sửa số lượng và ghi chú từng món | 备注随明细保存,长度上限 100 字符;同 SKU 不同备注拆行存储;无备注处展示「-」<br>Ghi chú lưu kèm chi tiết, tối đa 100 ký tự; cùng SKU khác ghi chú tách thành dòng riêng; không có ghi chú hiện "-" | |
| P03 | Web-C/BE | 可不预选直接预约<br>Có thể đặt bàn không chọn món | 不选择任何商品直接提交预约<br>Gửi đặt bàn mà không chọn món nào | 创建成功,预选明细为空;预选不影响桌台可用性与时段冲突判定<br>Tạo thành công, không có món chọn trước; việc chọn món không ảnh hưởng đến bàn trống và kiểm tra trùng khung giờ | |
| P04 | BE | 非法预选拦截<br>Chặn món chọn trước không hợp lệ | 提交数量 0/非整数,或 SKU 不属于当前商户/已不可售的预选<br>Gửi món với số lượng 0/không nguyên, hoặc SKU không thuộc merchant/không còn bán | 后端返回 400 或对应业务错误,不创建预约<br>Backend trả 400 hoặc lỗi nghiệp vụ tương ứng, không tạo đặt bàn | |
| P05 | BE | 有规格商品默认规格快照<br>Snapshot quy cách mặc định cho món có quy cách | 预选一个有规格的商品并提交预约<br>Chọn trước một món có quy cách và gửi đặt bàn | 预选明细按该商品默认规格存快照(原价与售价双价存储);到店转入时按默认规格判定购物车行合并<br>Chi tiết lưu snapshot theo quy cách mặc định (lưu cả giá gốc và giá bán); khi chuyển vào giỏ, gộp dòng theo quy cách mặc định | |
| P06 | Web-C/BE | 双价展示+到店价提示<br>Hiển thị 2 giá + ghi chú giá tại quán | 打开本人预约详情 R6 与分享页 R7 查看预选区<br>Mở chi tiết R6 và trang chia sẻ R7 xem khu vực món chọn trước | 每行展示划线原价+售价快照、数量、备注、小计与预估合计;页面标注「实际以到店时菜单价格为准」;R6 预选区展示「尚未加入购物车/已加入购物车」状态<br>Mỗi dòng hiện giá gốc gạch ngang + giá bán snapshot, số lượng, ghi chú, tạm tính và tổng ước tính; có dòng chú thích "Giá thực tế theo menu tại quán khi đến"; R6 hiện trạng thái "Chưa thêm vào giỏ/Đã thêm vào giỏ" | |
| P07 | BE | 价格快照不随菜单改价<br>Snapshot giá không đổi theo menu | 预约创建后修改菜单价格,客户在到店前再打开预约详情<br>Sau khi tạo đặt bàn, đổi giá menu, khách mở lại chi tiết trước khi đến | 详情仍展示创建时的价格快照,不被当前菜单价格改写<br>Chi tiết vẫn hiện snapshot giá lúc tạo, không bị giá menu hiện tại ghi đè | |

## 七、到店转入购物车 / Chuyển món vào giỏ hàng khi đến quán

| 编号 STT | 端 | 用例 Tên case | 操作步骤 Các bước | 预期结果 Kết quả mong đợi | 结果 KQ |
|---|---|---|---|---|---|
| T01 | Web-C/BE | 弹窗确认转入<br>Popup xác nhận chuyển món | 一笔当日「已确认」预约含 2 个有效预选商品,桌台购物车已有 1 条同 SKU 行;客户扫该桌台二维码进入点餐页,在弹窗 P3 点「加入购物车」<br>Đặt bàn hôm nay "Đã xác nhận" có 2 món hợp lệ, giỏ của bàn đã có 1 dòng cùng SKU; khách quét mã bàn vào trang gọi món, ở popup P3 bấm "Thêm vào giỏ" | 点餐页弹出 P3「添加预选菜品?」含明细+小计;确认后同 SKU 行数量合并增加、其余商品新增购物车行,价格取转入时菜单价;转入行备注在目标行无备注时写入、已有备注保留不覆盖;不生成订单、不触发厨打、不扣库存;详情预选区状态变「已加入购物车」<br>Trang gọi món hiện popup P3 "Thêm món đã chọn trước?" kèm chi tiết + tạm tính; sau xác nhận: dòng cùng SKU gộp tăng số lượng, món khác thêm dòng mới, giá theo menu lúc chuyển; ghi chú ghi vào nếu dòng đích chưa có, đã có thì giữ nguyên; không tạo đơn, không in bếp, không trừ kho; chi tiết đổi trạng thái "Đã thêm vào giỏ" | |
| T02 | Web-M/BE | 店员到店操作不触发转入<br>Thao tác đến quán của nhân viên không kích hoạt | 店员在商家端对一笔含预选商品的预约执行到店操作<br>Nhân viên thực hiện thao tác "Đến quán" trên Web-M cho đặt bàn có món chọn trước | 预约状态变「已到店」;预选明细不发生转入,桌台购物车无变化<br>Trạng thái chuyển "Đã đến"; món chọn trước không được chuyển, giỏ của bàn không đổi | |
| T03 | Web-C/BE | 部分不可售跳过<br>Bỏ qua món không còn bán | 预选明细中 1 个 SKU 已下架、另 1 个仍可售,客户在弹窗确认转入<br>Trong món chọn trước có 1 SKU đã gỡ bán, 1 SKU còn bán; khách xác nhận chuyển | 可售商品进入购物车;已下架商品被跳过,转入结果返回该商品与原因,点餐页 Toast 展示「已将 N 道预选菜品加入购物车」<br>Món còn bán vào giỏ; món đã gỡ bị bỏ qua, kết quả trả kèm món và lý do, trang gọi món hiện Toast "Đã thêm N món chọn trước vào giỏ" | |
| T04 | Web-C/BE | 转入幂等<br>Chuyển món idempotent | 同一预约完成转入后,客户再次进入该桌点餐页/重复点击/重试接口<br>Sau khi đã chuyển, khách vào lại trang gọi món/bấm lặp/retry API | 弹窗不再出现;接口按幂等成功返回;购物车数量不重复增加、不新增重复商品行<br>Popup không hiện lại; API trả thành công idempotent; số lượng giỏ không tăng thêm, không có dòng trùng | |
| T05 | BE | 部分失败重试只补未成功项<br>Retry chỉ bù món chưa chuyển | 首次转入时 1 个商品因临时不可售被跳过,之后恢复可售,客户再次进入点餐页触发转入<br>Lần đầu 1 món tạm hết bán bị bỏ qua, sau khi có bán lại khách vào lại trang để chuyển tiếp | 仅补入此前未成功的 1 个商品;首次已成功商品数量不重复增加<br>Chỉ thêm món chưa chuyển thành công lần trước; món đã chuyển không tăng lại | |
| T06 | Web-C/BE | 「跳过」语义<br>Ngữ nghĩa nút "Bỏ qua" | 客户在弹窗点「跳过」,之后再次进入该桌点餐页<br>Khách bấm "Bỏ qua" ở popup, sau đó vào lại trang gọi món của bàn đó | 本次不转入;下次进入弹窗再次出现,预选明细仍未转入;预约完成/取消/爽约后弹窗不再出现<br>Lần này không chuyển; lần sau vào lại popup hiện lại, món vẫn chưa chuyển; sau khi đặt bàn hoàn thành/hủy/không đến thì popup không hiện nữa | |
| T07 | BE | 取消/爽约后不得转入<br>Hủy/không đến thì không được chuyển | 含预选的「已确认」预约被取消或判定爽约后,客户扫该桌码进入点餐页<br>Đặt bàn "Đã xác nhận" có món chọn trước bị hủy hoặc đánh dấu không đến, khách quét mã bàn vào trang gọi món | 点餐页不出现弹窗;预选明细仅保留在预约记录中供查看,不占库存、不生成购物车行;商家端在转入前取消预约同样不转入<br>Không hiện popup; món chọn trước chỉ còn để xem trong bản ghi đặt bàn, không giữ kho, không tạo dòng giỏ; Web-M hủy trước khi chuyển cũng không chuyển | |
| T08 | BE | 写入失败可重试<br>Lỗi ghi giỏ có thể retry | 模拟转入过程中购物车写入发生系统异常<br>Mô phỏng lỗi hệ thống khi ghi giỏ hàng trong lúc chuyển | 接口返回 `CART_WRITE_FAILED` 与失败明细,预选明细保持未转入;客户下次进入可再次尝试,重试成功后购物车只增加一次<br>API trả `CART_WRITE_FAILED` kèm chi tiết lỗi, món vẫn chưa chuyển; lần sau khách thử lại, thành công chỉ cộng vào giỏ một lần | |

## 八、商家端列表与详情 / Web-M: Danh sách & chi tiết

| 编号 STT | 端 | 用例 Tên case | 操作步骤 Các bước | 预期结果 Kết quả mong đợi | 结果 KQ |
|---|---|---|---|---|---|
| W01 | Web-M | M3 来源列<br>Cột "Nguồn" (M3) | 客户自助创建一笔预订后,店员打开预订列表<br>Sau khi khách tự tạo đặt bàn, nhân viên mở danh sách đặt bàn | 列表新增「来源」列,Tag 区分「客户自助 / 代客预订」;该记录可正常执行详情、到店、编辑、取消等既有操作<br>Danh sách có thêm cột "Nguồn", tag phân biệt "Khách tự đặt / Đặt hộ"; bản ghi thao tác bình thường: chi tiết/đến quán/sửa/hủy | |
| W02 | Web-M | 客户取消同步可见<br>Đồng bộ khi khách hủy | 客户在 C 端取消一笔自助预订,店员查看列表与操作日志<br>Khách hủy đặt bàn tự đặt trên Web-C, nhân viên xem danh sách và nhật ký | 列表状态=已取消,日志含取消记录,来源标识保持「客户自助」<br>Trạng thái = Đã hủy, nhật ký có bản ghi hủy, tag nguồn vẫn là "Khách tự đặt" | |
| W03 | Web-M | M4 预选菜品卡(只读)<br>Thẻ món chọn trước ở M4 (chỉ đọc) | 打开一笔含 3 个预选商品的客户自助预订详情弹窗<br>Mở popup chi tiết của đặt bàn tự đặt có 3 món chọn trước | 展示来源「顾客自助预订」与「顾客预选菜品」卡片:表格(菜品/数量/划线原价+单价/金额/备注,无备注显「-」)+ 小计 + 转入状态标记(尚未加入购物车/已加入购物车);含脚注「实际以到店时菜单价格为准」「到店后自动加入购物车」;员工仅可查看,无修改预选明细的入口<br>Hiện nguồn "Khách tự đặt" và thẻ "Món khách chọn trước": bảng (món/số lượng/giá gốc gạch ngang + đơn giá/thành tiền/ghi chú, không ghi chú hiện "-") + tạm tính + nhãn trạng thái chuyển (Chưa thêm vào giỏ/Đã thêm vào giỏ); có chú thích cuối; nhân viên chỉ xem, không có nút sửa | |
| W04 | Web-M | 操作记录预选摘要<br>Tóm tắt món chọn trước trong nhật ký thao tác | 查看该预订的操作记录<br>Xem nhật ký thao tác của đặt bàn đó | 含创建条目并附预选摘要「预选菜品:N道菜 · 合计」<br>Có bản ghi tạo kèm tóm tắt "Món chọn trước: N món · tổng cộng" | |
