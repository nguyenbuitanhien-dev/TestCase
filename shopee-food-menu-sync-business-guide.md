# ShopeeFood 菜单同步 业务说明书(测试向)
# Hướng dẫn nghiệp vụ đồng bộ menu ShopeeFood (dành cho kiểm thử)

> 适用环境:scp 测试环境 | 配套文档:手工测试用例 `docs/test-reports/shopee-food-uat-manual-testcases-2026-11-16.md`、规格书 `docs/specs/shopee-food-api-integration.md`
> 本文为中越双语,每个小节先中文后越南语。

---

## 1. 功能全景 / Tổng quan

**中文**

我方系统(POS/Web 商家端)与 ShopeeFood(下称 SPF)对接,核心是「**菜单以我为准,单向推送**」:

```
本地菜品库 →(从菜品库添加)→ ShopeeFood 渠道菜单 →(完整同步)→ SPF 门店菜单
```

订单方向相反:客人在 SPF 下单 → SPF 推送给我方 → POS 接单/拒单 → 回传状态给 SPF。

**Tiếng Việt**

Hệ thống của ta (POS/Web cho merchant) kết nối với ShopeeFood (gọi tắt SPF), cốt lõi là「**menu lấy phía ta làm chuẩn, đẩy một chiều**」:

```
Thư viện món nội bộ →(Thêm từ thư viện món)→ Menu kênh ShopeeFood →(Đồng bộ toàn bộ)→ Menu cửa hàng SPF
```

Đơn hàng đi chiều ngược lại: khách đặt trên SPF → SPF đẩy về hệ thống → POS tiếp nhận/từ chối → trả trạng thái về SPF.

---

## 2. 数据从哪来(渠道菜单准备)/ Dữ liệu đến từ đâu (chuẩn bị menu kênh)

**中文**

- 同步的数据源**不是订单管理**,是「ShopeeFood 商品管理」页(POS:设置→ShopeeFood 商品管理;Web:菜品管理→ShopeeFood)里的**渠道菜单**。
- 渠道菜品 = 通过「**从菜品库添加**」从本地菜品库选进 SPF 渠道、且当前**上架中**的菜品;下架菜不会进推送。
- 每道渠道菜必须有「**SPU 编码**」(spu_code),它会被用作 SPF 侧的合作方菜品 ID(partner_dish_id)。**没填编码的菜同步会失败**,新菜进渠道前务必先填。
- 同步内容 = 分类 → 菜品(越语名称/价格/图片/售罄状态)→ 被菜品引用的加料组和加料 → 门店营业时间(支持一天多时段)。

**Tiếng Việt**

- Nguồn dữ liệu đồng bộ **không phải** quản lý đơn hàng, mà là **menu kênh** trong trang「Quản lý sản phẩm ShopeeFood」(POS: Cài đặt → Quản lý sản phẩm ShopeeFood; Web: Quản lý món → ShopeeFood).
- Món trong kênh = các món được chọn từ thư viện món nội bộ qua nút「**Thêm từ thư viện món**」và đang **mở bán**; món đã hạ sẽ không được đẩy.
- Mỗi món trong kênh bắt buộc có「**Mã SPU**」(spu_code) — mã này làm ID đối tác của món bên SPF (partner_dish_id). **Món thiếu mã sẽ đồng bộ thất bại**, nhớ điền trước khi thêm vào kênh.
- Nội dung đồng bộ = nhóm món → món (tên tiếng Việt/giá/ảnh/trạng thái hết hàng) → nhóm topping và topping được món tham chiếu → giờ mở cửa của cửa hàng (hỗ trợ nhiều khung giờ trong ngày).

---

## 3. 完整同步的工作方式 / Cách hoạt động của đồng bộ toàn bộ

**中文**

点「完整同步」后,系统按以下步骤工作:

1. 后端把当前渠道菜单组装成 SPF 官方菜单格式(Menu JSON v2);
2. 通知 SPF「菜单已更新,来取」(menu.sync);
3. **SPF 主动回拉**我方的菜单接口,取走完整菜单;
4. SPF 异步处理(建同步任务),我方每 30 秒轮询任务状态,直到成功或失败;
5. 完成后弹出结果弹窗:**已同步 / 已删除 / 失败** 三项计数。

两个要点:

- **全量覆盖语义**:SPF 门店上本次没推送的菜品会被**删除**(「已删除」计数就是这部分)。同步前请确认渠道菜单就是要售卖的完整菜单。
- SPF 不区分新建/更新,所以结果只有 3 项计数,没有「新建/更新」拆分。

**Tiếng Việt**

Sau khi bấm「Đồng bộ toàn bộ」, hệ thống chạy theo các bước:

1. Backend đóng gói menu kênh hiện tại theo định dạng chính thức của SPF (Menu JSON v2);
2. Thông báo cho SPF「menu đã cập nhật, hãy lấy」(menu.sync);
3. **SPF chủ động gọi ngược** lại API menu của ta để lấy toàn bộ menu;
4. SPF xử lý bất đồng bộ (tạo tác vụ), ta cứ 30 giây hỏi trạng thái một lần cho tới khi thành công hoặc thất bại;
5. Xong thì hiện popup kết quả: **Đã đồng bộ / Đã xóa / Lỗi** — ba chỉ số.

Hai điểm cần nhớ:

- **Ngữ nghĩa ghi đè toàn bộ**: món nào trên SPF không có trong lần đẩy này sẽ bị **xóa** (chỉ số「Đã xóa」chính là phần này). Trước khi đồng bộ hãy chắc menu kênh chính là menu bán đầy đủ.
- SPF không phân biệt tạo mới/cập nhật, nên kết quả chỉ có 3 chỉ số, không tách「tạo mới/cập nhật」.

---

## 4. 失败与重试 / Xử lý lỗi và thử lại

**中文**

**失败怎么看**:结果弹窗或菜品 Tab 点「查看原因」,弹窗显示 SPF 返回的失败原因原文(如 "price cannot exceed 10 million VND")。常见原因:

| 原因 | 处理 |
|---|---|
| 价格超过 1000 万越南盾(SPF 硬上限) | 改价到上限内或下架后重试 |
| 菜品未填 SPU 编码 | 补编码后重试 |
| 菜品/分类未映射 | 跑映射对账(本期由技术侧脚本执行) |

**重试三档**:

| 操作 | 范围 |
|---|---|
| 单行「查看原因 → 重试」 | 仅该对象 |
| 勾选多行「批量重试失败」 | 仅勾选中的失败项(勾选里没有失败项会提示) |
| 结果弹窗「重新执行完整同步」 | 仅上次任务的失败项 |

「重新执行完整同步」**不是全量重发**:系统提交上次任务号(taskId),后端从该任务 SPF 返回的失败明细(`failed_items_detail`)中取出失败对象,组一份**只含失败项**的新任务重新提交,生成新任务号。

**Tiếng Việt**

**Xem lỗi ở đâu**: trong popup kết quả hoặc tab Món, bấm「Xem lý do」— popup hiện nguyên văn lỗi do SPF trả về (ví dụ "price cannot exceed 10 million VND"). Nguyên nhân thường gặp:

| Nguyên nhân | Xử lý |
|---|---|
| Giá vượt 10 triệu VND (giới hạn cứng của SPF) | Đổi giá trong giới hạn hoặc hạ món rồi thử lại |
| Món chưa có Mã SPU | Điền mã rồi thử lại |
| Món/nhóm chưa mapping | Chạy đối soát mapping (giai đoạn này do kỹ thuật chạy script) |

**Ba mức thử lại**:

| Thao tác | Phạm vi |
|---|---|
|「Xem lý do → Thử lại」từng dòng | Chỉ đối tượng đó |
| Chọn nhiều dòng →「Thử lại mục lỗi」 | Chỉ các mục lỗi đã chọn (nếu không có mục lỗi sẽ báo) |
| Popup kết quả →「Chạy lại đồng bộ toàn bộ」 | Chỉ các mục lỗi của lần chạy trước |

「Chạy lại đồng bộ toàn bộ」**không đẩy lại toàn bộ**: hệ thống gửi taskId cũ, backend lấy các đối tượng lỗi từ `failed_items_detail` của lần đó, tạo tác vụ mới **chỉ gồm các mục lỗi** và gửi lại, sinh taskId mới.

---

## 5. 进度与状态显示 / Hiển thị tiến trình và trạng thái

**中文**

**进度条**:同步中顶部状态条显示「**x% · 任务 #编号**」。注意当前进度只有两个端点:开始时 0%,SPF 任务结束跳到 100%——**没有中间递增**(SPF 异步执行,不提供中间进度)。任务跑 2~3 分钟期间停在 0% 属正常,**不是卡死**,等结果弹窗即可。

**菜品同步状态字典**(菜品 Tab「同步状态」列):

| 状态 | 含义 |
|---|---|
| 已同步(绿) | 已推送到 SPF 且映射已建立 |
| 待同步 | 渠道菜,还没进过成功同步 |
| 失败(红) | 最近同步被 SPF 拒收(原因见「查看原因」) |
| 未映射(灰) | 没有映射记录(未推过/被拒没建档) |
| 分类未映射 | 菜品所属分类缺映射(失败优先显示) |

**审核状态列**:加料/菜品可能被 SPF 审核——待审核显「刷新」(点按实时查询),已拒绝显「查看原因」。

**Tiếng Việt**

**Thanh tiến trình**: khi đang đồng bộ, thanh trạng thái hiển thị「**x% · Tác vụ #số**」. Lưu ý tiến trình hiện chỉ có hai mốc: 0% lúc bắt đầu, nhảy 100% khi tác vụ SPF kết thúc — **không tăng dần ở giữa** (SPF chạy bất đồng bộ, không trả tiến độ trung gian). Tác vụ chạy 2~3 phút mà % dừng ở 0 là bình thường, **không phải treo**, chờ popup kết quả.

**Từ điển trạng thái đồng bộ món** (cột「Trạng thái đồng bộ」tab Món):

| Trạng thái | Ý nghĩa |
|---|---|
| Đã đồng bộ (xanh) | Đã đẩy lên SPF và mapping đã có |
| Chờ đồng bộ | Món trong kênh, chưa từng đồng bộ thành công |
| Lỗi (đỏ) | Lần đồng bộ gần nhất bị SPF từ chối (xem「Xem lý do」) |
| Chưa mapping (xám) | Chưa có bản ghi mapping (chưa đẩy/bị từ chối chưa tạo hồ sơ) |
| Nhóm chưa mapping | Nhóm của món thiếu mapping (ưu tiên hiển thị lỗi) |

**Cột trạng thái duyệt**: topping/món có thể bị SPF duyệt — chờ duyệt hiện「Làm mới」(bấm để tra ngay), bị từ chối hiện「Xem lý do」.

---

## 6. 增量同步(日常变化)/ Đồng bộ tăng dần (thay đổi hằng ngày)

**中文**

- 菜品设**售罄**(可设时长如 2 小时)→ 同步到 SPF 对应菜售罄;恢复售卖同步恢复。
- 菜品改价/上下架 → 走增量接口或下次完整同步覆盖。
- 加料售罄同理(SPF 侧加料状态联动)。

**Tiếng Việt**

- Đặt món **hết hàng** (có thể kèm thời lượng, ví dụ 2 giờ) → đồng bộ trạng thái hết hàng sang SPF; mở bán lại sẽ đồng bộ khôi phục.
- Đổi giá/hạ món → đi qua API tăng dần hoặc lần đồng bộ toàn bộ kế tiếp.
- Topping hết hàng tương tự (trạng thái topping bên SPF liên động).

---

## 7. 相关数据表(排查用,技术向)/ Bảng dữ liệu liên quan (tra cứu kỹ thuật)

| 表 | 内容 |
|---|---|
| `ovs_spf_menu_mapping` | 菜品/分类/加料映射与同步状态 |
| `ovs_spf_sync_task` | 同步任务:状态/计数/失败明细 |
| `ovs_spf_api_audit` | 出站调用报文(敏感字段已脱敏) |
| `ovs_spf_order_event` | 订单 webhook 事件与处理结果 |
