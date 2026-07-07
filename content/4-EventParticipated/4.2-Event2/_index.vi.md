---
title: "Event 2"
date: 2026-05-23
weight: 2
chapter: false
pre: " <b> 4.2. </b> "
---


# Bài thu hoạch "FCAJ Community Day"

**Thời gian:** 09:00, ngày 23/05/2026

### Mục Đích Của Sự Kiện

- Kéo các AWS community builder, kỹ sư đang đi làm và sinh viên lại với nhau để trao đổi kinh nghiệm thực chiến về AI, kiến trúc cloud, và cách các ứng dụng hiện đại thực sự được đưa vào sản xuất
- Đi sâu vào khía cạnh thực tế, đôi khi hơi khó chịu, của việc xây dựng với LLM — từ những điểm bất ổn định cho tới kỷ luật context engineering
- Đưa lên sân khấu những case study thật từ production và hackathon của chính cộng đồng, chứ không phải các ví dụ marketing được đánh bóng
- Nhìn kỹ vào Amazon CloudFront như một nền tảng cho chi phí dự đoán được, bảo mật và hiệu năng ở tầng edge

### Danh Sách Diễn Giả

- **Duc Dao** – Solution Architect, Cloud Kinetics — *Non-Determinism of "Deterministic" LLM Settings*
- **Vy Lam** – Senior Business Systems Analyst, VPBank — *Enterprise-Grade Multi-Agent System: The Case of Startup Credit Scoring*
- **Pham Nguyen Hai Anh** – AWS Community Builder, G-AsiaPacific Vietnam — *Friendly AI Assistant w/ Amazon Quick*
- **Nguyen Tuan Thinh** – DevOps Engineer, First Cloud AI Journey — *From Edge to Origin: CloudFront as Your Foundation*
- **Tinh Truong** – Platform Engineer, GoTymeX — *Context Is Everything*
- **Team VIB** – Đội thi LotusHacks 2026 — *UTMorpho: Building UTMorpho from Idea to Reality*

### Nội Dung Nổi Bật

#### Tính không ổn định của "Deterministic" LLM Settings

Duc Dao mở đầu bằng một phát hiện âm thầm lật đổ một điều mà hầu hết chúng ta vẫn mặc định là đúng: đặt `temperature=0` được cho là sẽ đảm bảo cùng một output ở mọi lần chạy, nhưng nghiên cứu của Atil và cộng sự (2024) cho thấy **không một model nào trong số được kiểm thử thực sự cho ra output nhất quán** trên toàn bộ các task. Độ chính xác dao động tới 15%, và khoảng cách giữa lần chạy tốt nhất và tệ nhất lên tới 70% — những con số khó mà khớp với từ "deterministic" (tất định).

Nguyên nhân gốc hoá ra lại tầm thường hơn nhiều so với một "tính khí" riêng của model: **phép toán số thực dấu phẩy động trên GPU vốn không có tính kết hợp (non-associative)**, cộng thêm việc các request được xử lý theo **batch** trong lúc inference, nên kết quả chính xác cho request của bạn âm thầm phụ thuộc vào việc có những request nào khác đang nằm chung batch tại thời điểm đó. Các giải pháp giảm thiểu mà Duc đưa ra khá thực dụng chứ không cầu kỳ: chạy cùng một request nhiều lần rồi lấy đa số (majority voting), ràng buộc output theo một định dạng có cấu trúc, tự host model khi cần toàn quyền kiểm soát tầng inference, và quan trọng có lẽ nhất là thiết kế toàn bộ logic ở downstream để coi output của LLM là **xác suất chứ không phải tất định** ngay từ đầu. Một mẹo nhỏ nhưng hữu ích anh chia sẻ: `temp=0.1` thường an toàn hơn việc set cứng `temp=0`, vì nó tránh được tình trạng model bị mắc kẹt trong vòng lặp lặp lại.

#### Enterprise-Grade Multi-Agent System — Credit Scoring cho Startup

Phần chia sẻ của Vy Lam bắt đầu từ một sự lệch pha rất dễ bỏ qua cho đến khi bạn nhìn thẳng vào nó: các mô hình chấm điểm tín dụng truyền thống được xây dựng dựa trên hơn ba năm lịch sử tài chính và báo cáo theo chuẩn — trong khi đó lại chính xác là thứ một startup không có, thay vào đó chỉ có 6 đến 18 tháng tăng trưởng nóng cùng dữ liệu lộn xộn, phi cấu trúc.

Theo chị, một agent AI đơn lẻ sẽ gãy trước bài toán kiểu này — nó bị yêu cầu ôm cùng lúc chuyên môn đa lĩnh vực, cân bằng các mục tiêu xung đột nhau, và vẫn phải giữ được khả năng audit, và dù về mặt kỹ thuật nó "chạy được", output vẫn không đủ tin cậy cho một quyết định có rủi ro tài chính thật. Giải pháp thay thế chị đề xuất là một **virtual credit committee** (uỷ ban tín dụng ảo): một tập hợp các agent chuyên biệt (Financial, Market, Team, Risk, Compliance) phối hợp dưới một Manager agent để đi đến một điểm số đồng thuận, một xếp hạng rủi ro, một mức độ tin cậy, và một audit trail đầy đủ đứng sau mỗi quyết định. Chị khung hoá tư duy enterprise-grade dựa trên sáu trụ cột — **security, data governance, network, operations, human factors, và compliance** — và minh hoạ bằng con số cụ thể: xử lý nhanh hơn 95% và ROI 3 năm đo được đạt 200-300%.

#### Friendly AI Assistant với Amazon Quick

Phần trình bày của Pham Nguyen Hai Anh nhắm thẳng vào một nỗi bực bội quen thuộc với người dùng doanh nghiệp: phần lớn thời gian trong ngày của họ bị ngốn bởi các tác vụ lặp lại, đòi hỏi phải kéo và đối chiếu thông tin nằm rải rác trên nhiều hệ thống khác nhau.

Thông điệp anh đưa ra cho **Amazon Quick Suite** là nó gom tất cả vào một lớp duy nhất, có quản trị và bảo mật — kết hợp dữ liệu công ty, kiến thức chung của thế giới, hơn 40 data connector, và các khả năng agentic (automation flow, dashboard, chat, research) đứng sau một bộ guardrail và access control duy nhất, thay vì phải ghép nối một đống công cụ rời rạc lại với nhau. Đoạn demo gây ấn tượng nhất là một PM assistant tự động soạn biên bản họp, gửi email cho các stakeholder liên quan, và tự lên lịch cho cuộc họp tiếp theo.

#### From Edge to Origin: CloudFront as Your Foundation

Phần chia sẻ của Nguyen Tuan Thinh bắt đầu từ một vấn đề mà bất kỳ ai từng vận hành một dịch vụ công khai đều từng cảm nhận: hoá đơn CDN tính theo usage vốn nổi tiếng khó dự đoán, có thể dao động từ vài xu cho đến tăng vọt lên hàng trăm nghìn đô chỉ vì traffic viral hoặc bị tấn công.

Anh đi qua câu trả lời của AWS cho sự khó lường đó — các **gói CDN và bảo mật giá cố định** mới (ra mắt tháng 11/2025) gộp CloudFront, WAF, chống DDoS, Route 53 và logging CloudWatch vào một mức phí hàng tháng dễ đoán, thay vì một hoá đơn bất ngờ theo usage. Ngoài chuyện giá cả, anh cũng lập luận rằng CloudFront cải thiện đồng thời bốn thứ: **chi phí** (nén dữ liệu, miễn phí data transfer từ AWS origin), **bảo mật** (TLS 1.3/mTLS, Origin Shield/OAC/VPC origin để giấu origin, signed URL), **độ bền vững** (multi-layer caching, origin failover, graceful degradation khi quá tải), và **hiệu năng** (HTTP/3, kết nối backbone bền vững, đẩy compute ra tận edge).

#### Context Is Everything

Phần trình bày của Tinh Truong nhìn lại một lời than phiền mà chắc ai cũng từng thốt ra ít nhất một lần — "AI trả lời dở quá" — và lập luận rằng thủ phạm thật sự gần như luôn là **context kém, không phải model yếu**. AI không thể tự suy ra bạn thực sự muốn gì hay đọc được suy nghĩ của bạn; nó chỉ có thể làm việc với những gì bạn đưa cho nó.

Anh liệt kê các kiểu sai lầm thường gặp: nhồi nhét mọi thứ vào một đoạn chat duy nhất rồi hy vọng model tự hiểu đâu là điều quan trọng (cái anh gọi là vấn đề "internet puller"), nhắc lại những thông tin model vốn đã có sẵn, và không bao giờ nêu rõ mục tiêu, ràng buộc, hay tiêu chí thành công là gì. Anh khung hoá sự tiến hoá của việc dùng AI như một chuỗi tiến triển — từ những **prompt** rời rạc, đến **context** có căn cứ, rồi đến **memory** lâu dài (vòng lặp store → retrieve → generate → learn) — và dùng case study "Second AI Brain" để minh hoạ điều đó trông như thế nào trong thực tế. Bài học thực tế anh để lại là một checklist bốn phần đơn giản, chạy qua trước khi hỏi AI bất kỳ điều gì: **mục tiêu, thông tin liên quan, ràng buộc, tiêu chí thành công**.

#### UTMorpho — Xây Dựng Sản Phẩm Trong 36 Giờ (LotusHacks 2026)

Team VIB khép lại ngày hôm đó bằng một buổi nhìn lại rất thật về hành trình LotusHacks 2026 của họ — bắt đầu từ trạng thái mà họ mô tả là "đầu óc trống rỗng" lúc đăng ký, và kết thúc với một sản phẩm đã hoàn thiện, UTMorpho, sau 36 giờ.

Họ không tô hồng những giai đoạn khó khăn: AI sinh ra nhiều hơn nhiều so với những gì họ cần, chạm giới hạn token vào những lúc không hề thuận lợi, và cảm giác kiệt sức len lỏi vào ngay trước giờ pitch. Điều giúp họ vượt qua không nằm nhiều ở việc viết prompt khéo léo, mà nằm ở việc tập trung chỉnh sửa và giao tiếp chặt chẽ hơn trong team. Bài học họ rút ra rơi vào cả khía cạnh động lực nhóm lẫn công nghệ: **sự bực bội thực sự tạo ra những ý tưởng thực sự**, nhưng có nhiều ý tưởng hơn không tự động đồng nghĩa với tốt hơn — sức bền và sự đồng bộ của cả team quan trọng không kém gì tech stack phía sau.

### Những Gì Học Được

#### Độ tin cậy của AI là bài toán thiết kế, không chỉ là bài toán model

- Các setting tất định như `temp=0` chỉ giảm bớt tính ngẫu nhiên chứ không xoá bỏ hoàn toàn — hệ thống cần được xây dựng để chấp nhận sự biến thiên đó, chứ không phải giả định nó không tồn tại.
- Với những quyết định rủi ro cao, trải rộng nhiều lĩnh vực, hoặc cần khả năng audit, kiến trúc multi-agent liên tục vượt trội hơn một agent đơn lẻ bị quá tải.

#### Context Engineering đang trở thành kỹ năng cốt lõi

- Context tốt hơn — chứ không đơn thuần là nhiều hơn — mới thực sự là yếu tố quyết định chất lượng output của AI.
- Sự dịch chuyển từ prompting, sang context system, rồi đến memory dài hạn đang dần trở thành khoảng kỹ năng tiếp theo mà developer cần lấp đầy.

#### Nền tảng hạ tầng vẫn quan trọng

- Các khả năng ở tầng edge của CloudFront (bảo mật, chi phí dự đoán được, hiệu năng) vẫn là hạ tầng nền tảng ngay cả trong một stack thiên nặng về AI.
- Tư duy enterprise-grade — security, governance, compliance — cần được thiết kế ngay từ ngày đầu tiên; gắn thêm sau đó sẽ không hiệu quả bằng.

### Ứng Dụng Vào Công Việc

- Mặc định coi output của LLM là xác suất — thêm majority-voting hoặc ràng buộc structured-output cho bất kỳ pipeline nào hiện đang dựa vào `temp=0` để đảm bảo tính nhất quán.
- Thử nghiệm một mô hình multi-agent cho bất kỳ workflow nào hiện đang bị dồn hết vào một agent quá tải.
- Chạy qua framework context bốn phần (mục tiêu, thông tin liên quan, ràng buộc, tiêu chí thành công) trước khi viết prompt cho bất cứ việc gì thực sự quan trọng.
- Đánh giá các gói giá cố định của CloudFront cho những dự án có traffic khó đoán, để tránh cú sốc hoá đơn bất ngờ về sau.
- Thử Amazon Quick Suite cho các workflow lặp lại trong công việc — ghi biên bản họp, cập nhật stakeholder, lên lịch — thay vì làm thủ công.

### Trải nghiệm trong event

Ngồi nghe **"FCAJ Community Day"** hoá ra là một góc nhìn khá tốt để thấy các mảng khác nhau trong cộng đồng AWS — kỹ sư doanh nghiệp, community builder, đội thi hackathon — đang thực sự áp dụng AI và nền tảng cloud vào thực tế như thế nào, chứ không chỉ trên lý thuyết. Vài điều đọng lại với mình sau sự kiện:

**Học từ chính người thực hành, không chỉ từ tài liệu chính thức.** Các diễn giả chia sẻ những phát hiện trực tiếp — như nghiên cứu về tính tất định của LLM — thay vì lặp lại các luận điểm mang tính marketing, và case study credit scoring cho startup cho thấy rất cụ thể cách một thiết kế multi-agent giải quyết bài toán audit thực tế trong một ngành bị quản lý chặt chẽ như ngân hàng.

**Thấy hạ tầng và các vấn đề AI va chạm nhau ngay trong cùng một cuộc trò chuyện.** Phiên về CloudFront là một lời nhắc hữu ích rằng nền tảng edge và networking không hề biến mất chỉ vì lộ trình của mọi người giờ đều xoay quanh AI, còn phần nói về context engineering giúp nhìn nhận lại prompting về bản chất là một vấn đề trải nghiệm sản phẩm, chứ không phải một mẹo dùng từ ngữ học một lần rồi dùng mãi.

**Nhìn thấy khía cạnh "người xây dựng" của cộng đồng, kể cả những phần không hoàn hảo.** Phần chia sẻ nhìn lại LotusHacks của Team VIB là một góc nhìn thật sự trung thực về cảm giác build sản phẩm trong 36 giờ từ bên trong — bao gồm cả những phần không suôn sẻ, chứ không chỉ bản demo đã được chỉn chu. Điều đó càng khẳng định rằng sự đồng bộ của team và sức bền quan trọng không kém kỹ năng kỹ thuật thuần tuý một khi bạn thực sự chịu áp lực thời gian.

**Điều mình rút ra từ tất cả những chuyện này:** độ tin cậy trong hệ thống AI phải được thiết kế chủ động — nó không phải thứ tự nhiên có được từ một setting temperature. Thiết kế context tốt và thiết kế hạ tầng tốt hoá ra đi theo đúng một nguyên tắc: cho hệ thống chính xác những gì nó cần, không thừa không thiếu. Và những sự kiện cộng đồng như thế này liên tục mang lại những bài học thực tế mà thật khó tìm thấy trong bất kỳ tài liệu chính thức nào.
