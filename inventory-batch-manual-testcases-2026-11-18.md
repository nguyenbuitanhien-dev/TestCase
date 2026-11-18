# 库存批次与先入先出(FIFO)手工测试用例(中越双语)
# Test case kiểm thử thủ công: Tồn kho theo lô & Nhập trước xuất trước - FIFO (Song ngữ Trung - Việt)

> 涉及端 Phạm vi:BE = vietnam-takeout-server;Web = vietnam-takeout-web 管理端 / Web = trang quản trị;POS = vietnam-takeout-pos。
> 前置条件 Điều kiện tiên quyết:批次 4 张表 DDL 已执行、新 jar 与新前端同批部署 / Đã chạy DDL 4 bảng lô, deploy đồng thời jar + frontend mới;测试账号归属同一销售单位(商户)/ Tài khoản test thuộc cùng một đơn vị bán hàng (merchant)。
> 术语 Thuật ngữ:批次 = lô;有效期 = hạn sử dụng (HSD);生产日期 = ngày sản xuất (NSX);先入先出 = nhập trước xuất trước (FIFO);盘盈/盘亏 = kiểm kê dư/thiếu;临期 = sắp hết hạn。
> 依据 Căn cứ:docs/specs/inventory-batch.md(AC-1~AC-27)。

## 一、批次生成(入库)/ Tạo lô khi nhập kho

| 编号 STT | 端 | 用例 Tên case | 操作步骤 Các bước | 预期结果 Kết quả mong đợi | 结果 KQ |
|---|---|---|---|---|---|
| A01 | BE+POS/Web | 正常入库生成批次<br>Tạo lô khi nhập kho thành công | 建入库单含商品 A 10 件,填生产日期 + 保质期 12 个月,确认「入库」<br>Tạo phiếu nhập có hàng A số lượng 10, điền NSX + hạn bảo quản 12 tháng, xác nhận "Nhập kho" | 生成 1 条批次:批次号/生产日期/有效期/入库数量/剩余数量/入库时间/来源入库单号齐全,剩余数量=10,有效期=生产日期+12 月;库存总量 +10<br>Tạo 1 lô đủ thông tin: mã lô/NSX/HSD/số lượng nhập/số lượng còn/thời gian nhập/mã phiếu nguồn; số lượng còn = 10, HSD = NSX + 12 tháng; tổng tồn +10 | |
| A02 | BE | 未填生产日期<br>Không điền NSX | 入库时不填生产日期直接确认入库<br>Nhập kho mà không điền NSX | 批次仍落库,有效期为 null,剩余数量=入库数量,不报错;该批次状态恒「正常」且不参与过期预警<br>Lô vẫn được lưu, HSD = null, số lượng còn = số lượng nhập, không lỗi; lô này luôn ở trạng thái "Bình thường" và không tham gia cảnh báo hết hạn | |
| A03 | Web/POS | 保质期单位换算<br>Quy đổi đơn vị hạn bảo quản | 同一生产日期,保质期分别选 365 日 / 12 月 / 1 年,查看「预计有效期」<br>Cùng 1 NSX, chọn hạn bảo quản lần lượt 365 ngày / 12 tháng / 1 năm, xem "HSD dự kiến" | 三种单位换算结果均正确(日历日/日历月/日历年);「预计有效期」只读自动计算;明细行同步展示各商品生产日期/有效期<br>Cả 3 cách quy đổi đều đúng (theo ngày/tháng/năm dương lịch); "HSD dự kiến" chỉ đọc, tự tính; dòng chi tiết hiện NSX/HSD từng hàng | |
| A04 | BE+POS/Web | 待入库不生成批次<br>Phiếu chờ nhập không tạo lô | 采购入库单仅保存制单,不点「入库」<br>Phiếu nhập mua hàng chỉ lưu, không bấm "Nhập kho" | 库存总量不变、不生成批次;点击「入库」动作生效时才生成批次(页面提示「确认入库时将为每项物料创建批次」)<br>Tổng tồn không đổi, không tạo lô; chỉ khi bấm "Nhập kho" mới tạo lô (trang có ghi chú "Khi xác nhận nhập kho sẽ tạo lô cho từng hàng hóa") | |
| A05 | BE | 入库数量为 0<br>Số lượng nhập = 0 | 明细入库数量为 0 时确认入库<br>Xác nhận nhập kho với số lượng chi tiết = 0 | 不生成剩余数量为正的批次,库存总量不变<br>Không tạo lô có số lượng dương, tổng tồn không đổi | |
| A06 | POS/Web | 确认弹窗与取消<br>Popup xác nhận & hủy | 确认入库前查看弹窗文案,点「取消」;再确认一次点「确定」<br>Trước khi xác nhận xem nội dung popup, bấm "Hủy"; sau đó thao tác lại và bấm "Đồng ý" | 弹窗文案含「将创建 N 个批次,总库存增加 X」「确认后将立即创建批次,批次日期不可修改」,N/X 与明细一致;取消则不落批次、不增库存;确定后正常生成<br>Popup hiện "Sẽ tạo N lô, tổng tồn tăng X" và "Sau xác nhận sẽ tạo lô ngay, ngày lô không thể sửa", N/X khớp chi tiết; Hủy thì không tạo lô, không tăng tồn; Đồng ý thì tạo bình thường | |

## 二、先入先出扣减(FIFO)/ Trừ kho FIFO

| 编号 STT | 端 | 用例 Tên case | 操作步骤 Các bước | 预期结果 Kết quả mong đợi | 结果 KQ |
|---|---|---|---|---|---|
| B01 | BE | 跨批次扣减<br>Trừ qua nhiều lô | 造数据:商品有批次 B1(入库最早,剩 5)、B2(剩 8);出库 10 件<br>Dữ liệu: hàng có lô B1 (nhập sớm nhất, còn 5), B2 (còn 8); xuất 10 | B1 剩余 = 0、B2 剩余 = 3,库存总量 = 3;任一批次不出现负数<br>B1 còn 0, B2 còn 3, tổng tồn = 3; không lô nào bị âm | |
| B02 | BE | 动用历史未分配余量<br>Dùng phần tồn lịch sử chưa phân bổ | 商品批次剩余之和 = 6、库存总量 = 10(含 4 件历史未分配),出库 10 件<br>Tổng còn các lô = 6, tổng tồn = 10 (gồm 4 tồn lịch sử chưa phân bổ), xuất 10 | 两批次剩余均扣至 0,差额从历史未分配余量扣减,库存总量 = 0,不出现负库存、不拦截<br>Các lô đều về 0, phần chênh trừ vào tồn chưa phân bổ, tổng tồn = 0, không âm kho, không chặn | |
| B03 | BE | 出库超总量<br>Xuất vượt tổng tồn | 商品库存总量 = 6,出库 10 件<br>Tổng tồn = 6, xuất 10 | 按既有库存不足口径处理,不超扣、不出现负库存<br>Xử lý theo logic thiếu hàng hiện hữu, không trừ quá, không âm kho | |

## 三、POS 出库/报损 FIFO 交互 / POS: Tương tác FIFO khi xuất kho/hao hụt

| 编号 STT | 端 | 用例 Tên case | 操作步骤 Các bước | 预期结果 Kết quả mong đợi | 结果 KQ |
|---|---|---|---|---|---|
| C01 | POS | 「扣减批次」列实时预览<br>Cột "Lô sẽ trừ" xem trước theo thời gian thực | POS 内部出库页,商品批次 B1(早,剩 5)、B2(剩 8):先录入出库数量 10,再改为 3<br>Trang xuất kho nội bộ POS, hàng có B1 (sớm, còn 5), B2 (còn 8): nhập số lượng xuất 10, rồi đổi thành 3 | 页顶固定提示条「将从最早入库批次自动扣减 — 先进先出 (FIFO)」;数量 10 时「扣减批次」列展示 B1、B2 两个批次号;改为 3 时仅展示 B1<br>Đầu trang có thanh thông báo cố định "Sẽ tự động trừ từ lô nhập sớm nhất — FIFO"; khi nhập 10, cột "Lô sẽ trừ" hiện B1, B2; khi đổi còn 3 chỉ hiện B1 | |
| C02 | POS | 超批次确认弹窗<br>Popup xác nhận khi vượt tổng lô | 批次剩余之和 6、库存总量 10,录入出库 10 并提交;分别测「取消」与「确认」<br>Tổng còn các lô = 6, tổng tồn = 10, nhập xuất 10 và gửi; thử cả "Hủy" và "Xác nhận" | 弹出确认框「确认创建出库单?批次总数量不足;系统将依次扣减至 0,确保库存不会出现负数。」;取消则不提交;确认后扣减成功(批次归 0、总量归 0,不负库存)<br>Popup hiện "Xác nhận tạo phiếu xuất? Tổng số lượng lô không đủ; hệ thống sẽ trừ dần về 0, đảm bảo tồn kho không âm."; Hủy thì không gửi; Xác nhận thì trừ thành công (lô về 0, tổng về 0, không âm) | |
| C03 | POS | 报损 FIFO + 上限拦截<br>Hao hụt FIFO + chặn vượt tối đa | 商品当前库存 100、批次 B1 剩 100:先报损 30,再尝试报损超过 100<br>Tồn hiện tại 100, lô B1 còn 100: hao hụt 30, rồi thử hao hụt quá 100 | 报损 30 后 B1 剩余 70、总量 70,生成报损/出库流水,报损详情可见批次扣减明细;输入超过 100 时被拦截,输入框提示「最大值 N」<br>Sau hao hụt 30: B1 còn 70, tổng 70, có bút toán hao hụt/xuất kho, chi tiết xem được lô đã trừ; nhập quá 100 bị chặn, ô nhập hiện "Giá trị tối đa N" | |

## 四、批次展示与导出 / Hiển thị lô & xuất Excel

| 编号 STT | 端 | 用例 Tên case | 操作步骤 Các bước | 预期结果 Kết quả mong đợi | 结果 KQ |
|---|---|---|---|---|---|
| D01 | Web/POS | 库存列表「批次」列<br>Cột "Lô" trong danh sách tồn kho | 打开库存列表页,查看有批次与无批次的商品行<br>Mở trang danh sách tồn kho, xem hàng có lô và không có lô | 有批次显示「N个批次」(可点击);无批次显示「-」;POS 顶部 tab(库存/入库/出库/损耗/创建申请)与「盘点」入口保持现状<br>Hàng có lô hiện "N lô" (bấm được); không có lô hiện "-"; trên POS các tab đầu trang (Tồn kho/Nhập/Xuất/Hao hụt/Tạo yêu cầu) và nút "Kiểm kê" giữ nguyên | |
| D02 | Web | 行内展开批次子表<br>Mở rộng dòng xem bảng lô con (Web) | Web 库存页展开一个有 3 条批次的商品;点「查看全部 N 个批次」<br>Ở Web mở rộng hàng có 3 lô; bấm "Xem tất cả N lô" | 子表展示 3 行,列含批次号/生产日期/有效期/入库数量/库存数量(剩余)/入库日期/入库单/状态;剩余为 0 的批次也展示(状态=已用完);「查看全部」打开批次抽屉<br>Bảng con 3 dòng, đủ cột: mã lô/NSX/HSD/số lượng nhập/số lượng tồn (còn)/ngày nhập/phiếu nhập/trạng thái; lô còn 0 vẫn hiện (trạng thái = Đã dùng hết); "Xem tất cả" mở drawer lô | |
| D03 | POS | 点击直接开抽屉<br>POS bấm mở thẳng drawer | POS 库存列表点「N个批次」<br>Trên POS bấm "N lô" | 直接打开批次抽屉(不做行内展开)<br>Mở thẳng drawer lô (không mở rộng dòng) | |
| D04 | Web/POS | 批次抽屉汇总 + 历史库存提示<br>Drawer lô: tổng hợp + ghi chú tồn lịch sử | 打开一个 总库存 2046.7、批次合计 1000 的商品的批次抽屉<br>Mở drawer lô của hàng có tổng tồn 2046.7, tổng các lô 1000 | 展示商品标识(名称·规格·单位)、总库存、批次数、批次合计;因 总库存 > 批次合计,展示提示「部分库存尚未分配至任何批次(历史库存)」;批次按入库时间升序<br>Hiện thông tin hàng (tên · quy cách · đơn vị), tổng tồn, số lô, tổng các lô; vì tổng tồn > tổng lô nên hiện ghi chú "Một phần tồn kho chưa được phân bổ vào lô nào (tồn kho lịch sử)"; lô sắp xếp theo thời gian nhập tăng dần | |
| D05 | Web/POS | 状态筛选与空态<br>Lọc theo trạng thái & trạng thái trống | 抽屉内按 全部/正常/临期/已过期/已用完 筛选,点重置;再打开一个无批次商品的抽屉<br>Trong drawer lọc Tất cả/Bình thường/Sắp hết hạn/Đã hết hạn/Đã dùng hết, bấm Đặt lại; rồi mở drawer của hàng không có lô | 筛选结果正确;无匹配时展示空态;无批次商品返回空列表不报错<br>Kết quả lọc đúng; không có kết quả thì hiện trạng thái trống; hàng không có lô trả danh sách rỗng, không lỗi | |
| D06 | Web | 库存导出双模式<br>Xuất Excel 2 chế độ (chỉ Web) | Web 分别执行「仅总库存」「总库存 + 批次明细」导出;检查 POS<br>Trên Web xuất lần lượt "Chỉ tổng tồn" và "Tổng tồn + chi tiết lô"; kiểm tra POS | 「总库存+批次明细」Excel 含「总库存」「批次明细」2 个工作表,批次明细行数与系统批次记录一致;POS 端不提供导出入口<br>File "Tổng tồn + chi tiết lô" có 2 sheet "Tổng tồn kho" và "Chi tiết lô", số dòng chi tiết khớp với bản ghi trên hệ thống; POS không có chức năng xuất | |

## 五、临期预警 / Cảnh báo sắp hết hạn

| 编号 STT | 端 | 用例 Tên case | 操作步骤 Các bước | 预期结果 Kết quả mong đợi | 结果 KQ |
|---|---|---|---|---|---|
| E01 | BE | 阈值内触发预警<br>Cảnh báo trong ngưỡng | 造批次:有效期 = 今天 + 5 天、剩余数量 > 0;商户预警阈值 7 天;执行预警定时任务<br>Tạo lô: HSD = hôm nay + 5 ngày, số lượng còn > 0; ngưỡng cảnh báo của merchant = 7 ngày; chạy job cảnh báo | 该商户生成一条系统消息(msgCategory=2),内容含商品、批次号、有效期、剩余数量<br>Merchant nhận 1 thông báo hệ thống (msgCategory=2), nội dung gồm tên hàng, mã lô, HSD, số lượng còn | |
| E02 | BE | 阈值外不预警 / 已过期仍预警<br>Ngoài ngưỡng không cảnh báo / quá hạn vẫn cảnh báo | 批次 A 有效期 = 今天 + 8 天(超阈值);批次 B 已过期且剩余 > 0;执行任务<br>Lô A: HSD = hôm nay + 8 ngày (vượt ngưỡng); lô B: đã quá hạn và còn > 0; chạy job | 批次 A 不生成消息;批次 B 生成预警消息<br>Lô A không tạo thông báo; lô B có thông báo cảnh báo | |
| E03 | BE | 剩余 0 / 有效期空不预警<br>Còn 0 / HSD trống không cảnh báo | 批次 C 剩余 = 0(无论是否过期);批次 D 有效期为 null;执行任务<br>Lô C còn = 0 (kể cả quá hạn); lô D HSD = null; chạy job | 两个批次均不生成预警消息<br>Cả 2 lô đều không tạo thông báo | |
| E04 | BE | 预警幂等<br>Cảnh báo không trùng lặp | 某临期批次已生成过预警,同一预警窗口内再次执行任务<br>Một lô sắp hết hạn đã có thông báo, chạy lại job trong cùng chu kỳ cảnh báo | 不重复生成消息(同一批次同一预警窗口仅一条)<br>Không tạo thêm thông báo (mỗi lô chỉ 1 thông báo trong cùng chu kỳ) | |
| E05 | Web | 临期预警设置(仅 Web)<br>Cài đặt cảnh báo (chỉ Web) | Web 库存通知页点「设置」:把提前预警天数从 7 改为 15 保存;再尝试输入 0 或负数;检查 POS<br>Ở trang Thông báo tồn kho trên Web bấm "Cài đặt": đổi số ngày cảnh báo trước từ 7 thành 15 và lưu; thử nhập 0 hoặc số âm; kiểm tra POS | 弹窗含说明「剩余数量 > 0 且有效期 ≤ 今天 + N 天的批次将触发预警」「同一批次在同一预警周期内仅提醒一次」,范围固定「当前销售单位」;改为 15 后有效期 ≤ 今天+15 天的批次触发预警;0/负数校验失败并提示;POS 不提供预警设置入口<br>Popup có mô tả "Lô có số lượng còn > 0 và HSD ≤ hôm nay + N ngày sẽ kích hoạt cảnh báo" và "Mỗi lô chỉ nhắc 1 lần trong cùng chu kỳ", phạm vi cố định "Đơn vị bán hàng hiện tại"; sau khi đổi 15, lô có HSD ≤ hôm nay + 15 ngày được cảnh báo; nhập 0/số âm báo lỗi; POS không có mục cài đặt cảnh báo | |
| E06 | Web | 库存通知页<br>Trang thông báo tồn kho (Web) | 造 12 条未读预警,打开「库存通知」页;点「标为已读」;点某条「查看」<br>Tạo 12 thông báo chưa đọc, mở trang "Thông báo tồn kho"; bấm "Đánh dấu đã đọc"; bấm "Xem" ở một dòng | 页头未读徽标显示 12;列表列:物料名称/批次号/消息内容(如「有效期预警(即将到期)」)/有效期/剩余;「标为已读」后徽标清零;「查看」跳转该商品的批次抽屉<br>Huy hiệu chưa đọc hiện 12; các cột: tên hàng/mã lô/nội dung (vd "Cảnh báo HSD (sắp hết hạn)")/HSD/số lượng còn; sau "Đánh dấu đã đọc" huy hiệu về 0; "Xem" nhảy tới drawer lô của hàng đó | |
| E07 | POS | 消息中心「系统通知」tab<br>Tab "Thông báo hệ thống" trong trung tâm tin nhắn (POS) | POS 打开消息中心,切到「系统通知」tab;执行标为已读与查看跳转<br>Mở trung tâm tin nhắn trên POS, chuyển sang tab "Thông báo hệ thống"; thử đánh dấu đã đọc và xem chi tiết | 「系统通知」tab 与「待处理订单」并列,各带未读数徽标;列表列与 Web 一致;支持「标为已读」、查看跳转批次抽屉;无预警设置入口<br>Tab "Thông báo hệ thống" nằm cạnh tab "Đơn chờ xử lý", mỗi tab có huy hiệu số chưa đọc; cột giống Web; hỗ trợ "Đánh dấu đã đọc" và xem nhảy tới drawer lô; không có mục cài đặt cảnh báo | |

## 六、盘库联动 / Liên kết kiểm kê

| 编号 STT | 端 | 用例 Tên case | 操作步骤 Các bước | 预期结果 Kết quả mong đợi | 结果 KQ |
|---|---|---|---|---|---|
| F01 | Web/POS | 盘库页差异与处理方式<br>Trang kiểm kê: chênh lệch & cách xử lý | 盘库页录入实际数量(覆盖盘盈、盘亏、无差异三类行)<br>Nhập số lượng thực tế ở trang kiểm kê (có dòng dư, dòng thiếu, dòng khớp) | 每行展示 库存数量/实际数量/差异(盈正亏负)/处理方式(系统自动:「盘盈 → 创建待补批次入库单」「盘亏 → 按先进先出扣减」);页尾汇总「盘盈 N 项 · 盘亏 N 项 · 无差异 N 项」<br>Mỗi dòng hiện tồn hệ thống/thực tế/chênh lệch (dư dương, thiếu âm)/cách xử lý tự động ("Dư → tạo phiếu nhập chờ bổ sung lô" / "Thiếu → trừ theo FIFO"); cuối trang tổng hợp "Dư N mục · Thiếu N mục · Không lệch N mục" | |
| F02 | BE+Web/POS | 盘盈生成待补批次入库单<br>Dư → tạo phiếu nhập chờ bổ sung lô | 商品系统库存 = 10、盘库实际 = 15(盈 5),提交盘库<br>Tồn hệ thống = 10, thực tế kiểm kê = 15 (dư 5), gửi phiếu kiểm kê | 提交前确认框含「盘盈将生成 N 张待补批次入库单」「盘亏将按先进先出扣减 X」「确认后无法撤销」;确认后生成差额 5、类型=盘点、状态=待补批次的入库单;此时库存总量仍为 10、不生成批次;入库单列表新增「待补批次」状态展示与筛选,操作入口为「入库」<br>Trước khi gửi có popup "Phần dư sẽ tạo N phiếu nhập chờ bổ sung lô" / "Phần thiếu sẽ trừ theo FIFO: X" / "Sau xác nhận không thể hoàn tác"; sau xác nhận tạo phiếu nhập chênh 5, loại = kiểm kê, trạng thái = chờ bổ sung lô; tổng tồn vẫn 10, chưa tạo lô; danh sách phiếu nhập có trạng thái "Chờ bổ sung lô" để lọc, nút thao tác là "Nhập kho" | |
| F03 | BE+Web/POS | 待补批次入库单填写并入库<br>Điền thông tin lô và nhập kho | 打开待补批次入库单,逐项填写生产日期(必填)+ 有效期(必填,直接选日期),点「入库」<br>Mở phiếu nhập chờ bổ sung lô, điền từng hàng NSX (bắt buộc) + HSD (bắt buộc, chọn ngày), bấm "Nhập kho" | 页面标识「待补批次」状态、来源=盘点、入库单号、操作人;每项展示 系统库存 → 实际(盘库值)、差额数量;批次号只读「系统自动生成」;入库后生成剩余数量 5 的新批次,库存总量 = 15<br>Trang hiện trạng thái "Chờ bổ sung lô", nguồn = kiểm kê, mã phiếu, người thao tác; mỗi hàng hiện tồn hệ thống → thực tế, số chênh; mã lô chỉ đọc "Hệ thống tự tạo"; sau khi nhập tạo lô mới còn 5, tổng tồn = 15 | |
| F04 | Web/POS | 必填校验<br>Kiểm tra trường bắt buộc | 待补批次入库单未填生产日期或有效期,点「入库」<br>Không điền NSX hoặc HSD, bấm "Nhập kho" | 校验失败,行内红字提示「请输入生产日期 / 请输入有效期」,不允许入库、不生成批次<br>Báo lỗi, dòng hiện chữ đỏ "Vui lòng nhập ngày sản xuất / Vui lòng nhập hạn sử dụng", không cho nhập kho, không tạo lô | |
| F05 | Web/POS | 批量设置日期<br>Đặt ngày hàng loạt | 在待补批次入库单点「批量设置日期」,填生产日期+有效期并应用<br>Ở phiếu chờ bổ sung lô, bấm "Đặt ngày hàng loạt", điền NSX + HSD và áp dụng | 所有明细行批量填充生产日期+有效期;填充后仍可逐行修改<br>Tất cả dòng được điền NSX + HSD; sau đó vẫn sửa được từng dòng | |
| F06 | Web/POS | 无保存草稿<br>Không lưu nháp | 填写一半内容后退出页面,再次进入该单<br>Điền một nửa nội dung rồi thoát trang, vào lại phiếu đó | 未提交的填写内容不保留;单据保持「待补批次」状态,可继续填写<br>Nội dung chưa gửi không được giữ lại; phiếu vẫn ở trạng thái "Chờ bổ sung lô", có thể điền tiếp | |
| F07 | BE+Web/POS | 盘亏按 FIFO 扣减<br>Thiếu → trừ theo FIFO | 商品有 B1(早,剩 8)、B2(剩 5);盘库实际 = 10(系统 13,亏 3),提交<br>Hàng có B1 (sớm, còn 8), B2 (còn 5); kiểm kê thực tế = 10 (hệ thống 13, thiếu 3), gửi | 从 B1 扣减 3,B1 剩 5,总量 = 10;生成对应出库/报损流水;盘库结果页展示盘盈生成的待补批次入库单(单号+商品+差额)与盘亏扣减明细「B1 · 扣减 3」<br>Trừ 3 từ B1, B1 còn 5, tổng = 10; tạo bút toán xuất/hao hụt tương ứng; trang kết quả kiểm kê hiện phiếu nhập chờ bổ sung lô phần dư (mã phiếu + hàng + chênh) và chi tiết trừ phần thiếu "B1 · trừ 3" | |
| F08 | BE | 盘亏跨批次扣减<br>Thiếu trừ qua nhiều lô | 盘亏数量大于最早批次剩余,提交盘库<br>Số lượng thiếu lớn hơn số còn của lô sớm nhất, gửi kiểm kê | 依次跨批次扣减直至满足差额;仍不足时按历史未分配余量口径扣减;任何批次剩余不为负<br>Trừ dần qua các lô đến khi đủ số thiếu; nếu vẫn thiếu thì trừ phần tồn lịch sử chưa phân bổ; không lô nào bị âm | |
| F09 | Web/POS | 已入库单详情批次列<br>Cột lô trong chi tiết phiếu đã nhập | 打开一张已入库的入库单详情<br>Mở chi tiết một phiếu nhập đã nhập kho | 明细表展示 批次号/生产日期/有效期 列<br>Bảng chi tiết có các cột mã lô/NSX/HSD | |
