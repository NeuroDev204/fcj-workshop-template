---
title: "Event 1"
date: 2026-05-09
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---


# Bài thu hoạch "FCAJ Community Day"

**Thời gian:** 09:00, ngày 09/05/2026

### Mục Đích Của Sự Kiện

- Nhìn lại cách AI đang âm thầm viết lại thói quen, công cụ và quy trình hàng ngày của developer — từ việc học, viết prompt, làm quen với một codebase mới, cho đến việc ra sản phẩm
- Đào sâu vào những kỹ thuật cụ thể, lặp lại được để nâng chất lượng output của LLM, thay vì kiểu chỉnh sửa thử-sai theo cảm tính như trước giờ
- Bóc tách xem "AI-ready" thực sự nghĩa là gì đối với một sinh viên mới ra trường trong thị trường tuyển dụng hiện tại
- Tìm hiểu một cách làm có cấu trúc, dựa trên nhiều agent, để phát triển phần mềm với AI — như một lựa chọn thay thế cho một phiên chat dài không có tổ chức

### Danh Sách Diễn Giả

- **Huynh Hoang Long** – Admin của FCAJ - *Addicted to learning like you're addicted to Social Media*
- **Nguyen Tuan Thinh** – DevOps/Cloud Engineer - *Automated Prompt Engineering: Enhancing LLM Output Quality*
- **Khang** – Solution Architect - *AI-Ready Freshers*
- **Thao** - Software development - *BMAD Method*

### Nội Dung Nổi Bật

#### Addicted to Learning Like You're Addicted to Social Media

Huynh Hoang Long mở đầu bằng một điều mà có lẽ ai cũng cảm nhận được nhưng ít khi gọi tên rõ ràng: mạng xã hội không "vô tình" gây nghiện — nó được thiết kế có chủ đích xoay quanh một vòng lặp chặt chẽ **trigger (kích hoạt) → action (hành động) → variable reward (phần thưởng biến thiên) → investment (đầu tư)**, và chẳng có lý do gì để vòng lặp đó chỉ dành riêng cho việc lướt mạng. Nó hoàn toàn có thể được hướng vào việc học.

Luận điểm cốt lõi của anh là ý chí không phải là chiến lược bền vững, vì bản chất nó thất thường — có ngày bạn tràn đầy động lực, có ngày thì không, và một thói quen chỉ sống được vào những ngày "sung sức" thì coi như không tồn tại. Thứ thực sự trụ vững lại chính là môi trường và vòng phản hồi xung quanh hành vi đó, chứ không phải kỷ luật cá nhân của người thực hiện.

Anh sau đó đi qua từng cơ chế mà ai cũng có thể áp dụng: một trigger hằng ngày (nhắc nhở, streak), một hành động được thiết kế đủ nhỏ để lặp lại mà không gây khó chịu, một phần thưởng đủ biến thiên để giữ sự hứng thú (một kiến thức mới, một chiến thắng nhỏ, sự công nhận từ cộng đồng), và một bước "đầu tư" — ghi chú lại, chia sẻ công khai — để âm thầm khoá bạn vào vòng lặp cho lần sau. Bài học anh để lại: hãy coi việc học đều đặn là một **bài toán thiết kế thói quen** mà bạn kỹ thuật hoá, chứ không phải bài toán kỷ luật mà bạn phải gồng mình vượt qua.

#### Automated Prompt Engineering: Enhancing LLM Output Quality

Phần trình bày của Nguyen Tuan Thinh chạm đúng vào một nỗi đau mà nhiều người trong chúng ta từng cảm nhận nhưng chưa gọi tên chính xác: chỉnh prompt thủ công theo kiểu thử-sai đơn giản là không thể scale, và điều đáng lo là chỉ một thay đổi nhỏ trong cách diễn đạt cũng có thể làm chất lượng và độ ổn định của output lệch đi đáng kể.

Giải pháp anh đề xuất là ngừng coi việc viết prompt như một "nghệ thuật" và bắt đầu coi nó như một quy trình kỹ thuật: xác định rõ chỉ số thành công ngay từ đầu, xây một tập dữ liệu đánh giá nhỏ, tạo ra nhiều phiên bản prompt ứng viên, chấm điểm từng ứng viên trên tập dữ liệu đó (kể cả dùng chính LLM để làm giám khảo chấm điểm), rồi tiếp tục cải thiện phiên bản đang thắng. Nhìn theo hướng đó, prompt engineering không còn là đoán mò mà trở thành một **quy trình có thể lặp lại và đo lường được** — nơi bạn có thể so sánh khách quan phiên bản prompt A với phiên bản B trước khi đưa bất kỳ cái nào lên production. Ý tưởng này cũng nối thẳng vào một chủ đề liên tục được nhắc lại xuyên suốt sự kiện: bạn không thể loại bỏ hoàn toàn sự biến thiên của LLM, nhưng hoàn toàn có thể giảm nó một cách có hệ thống.

#### AI-Ready Freshers

Phần chia sẻ của Khang nhắm thẳng vào những ai sắp ra trường bước vào thị trường tuyển dụng hiện tại, với một góc nhìn khá thẳng thắn: các công cụ AI đã đảm nhiệm phần lớn công việc lặp lại, mang tính khuôn mẫu mà trước đây các kỹ sư junior thường được tuyển vào để làm — nên việc "biết viết prompt" một mình nó không còn là điểm khác biệt nữa.

Điều anh cho rằng thực sự tách biệt một fresher "AI-ready" khỏi những người còn lại là một tổ hợp những thứ chẳng liên quan gì đến kỹ năng viết prompt — **nền tảng vững chắc** (cấu trúc dữ liệu, tư duy hệ thống, khả năng debug khi có sự cố), thói quen đánh giá phản biện output của AI thay vì chấp nhận nó một cách mù quáng, và sự tự tin thực sự khi làm việc trong các workflow có AI hỗ trợ (Copilot, Amazon Q Developer, và các công cụ tương tự). Lời khuyên thực tế của anh cho bất kỳ ai đang xây portfolio: kết hợp công cụ AI với kỷ luật kỹ thuật thực sự — testing, version control, code review — thay vì hoàn toàn dựa vào "vibe coding" và hy vọng nó trụ được.

#### BMAD Method

Thao khép lại phần trình bày bằng cách giới thiệu **BMAD (Breakthrough Method for Agile AI-Driven Development)** — một cách tổ chức việc phát triển phần mềm có AI hỗ trợ xoay quanh các vai trò agent chuyên biệt — Analyst, PM, Architect, Scrum Master, Developer, QA — mỗi agent phụ trách một phần việc rõ ràng, được định nghĩa cụ thể trong quy trình.

Thay vì ném mọi thứ vào một AI assistant đa năng duy nhất và hy vọng nó tự xoay xở cả việc lập kế hoạch lẫn thực thi cùng lúc, BMAD chủ động tách hai giai đoạn này ra riêng biệt: mỗi agent tạo ra một tài liệu có cấu trúc (một PRD, một tài liệu kiến trúc, một story file) được bàn giao gọn gàng cho agent tiếp theo. Kết quả, theo cách Thao mô tả, là việc phát triển bằng AI vẫn bám sát agile practice thay vì trôi dần vào hỗn loạn — khả năng truy vết tốt hơn, tính nhất quán cao hơn, và cả team vẫn đồng bộ xuyên suốt dự án thay vì mỗi người dựa vào một đoạn chat dài lê thê, không có cấu trúc.

### Những Gì Học Được

#### Phát triển bản thân cần thiết kế hệ thống, không chỉ cần ý chí

- Thói quen học tập đều đặn hoàn toàn có thể được kỹ thuật hoá giống hệt cách một product team thiết kế vòng lặp engagement.
- Những hành động nhỏ, lặp lại được, có phản hồi rõ ràng luôn thắng những mục tiêu to tát nhưng âm thầm trở nên bất khả thi.

#### Viết prompt và ứng dụng AI cần sự chặt chẽ, không chỉ dựa vào cảm tính

- Chất lượng prompt là thứ bạn đo lường và cải thiện có hệ thống — không phải thứ bạn tinh chỉnh theo cảm giác.
- "AI-ready" phần lớn nằm ở khả năng phán đoán và nền tảng vững, chứ không phải một danh sách mẹo viết prompt.

#### Có cấu trúc luôn tốt hơn dùng AI một cách tùy hứng

- BMAD là minh chứng cụ thể cho việc trao cho AI agent vai trò và điểm bàn giao rõ ràng sẽ cho kết quả nhất quán hơn nhiều so với một assistant không có cấu trúc cố ôm đồm mọi thứ.
- Đây thực chất chỉ là một ví dụ của một quy luật lớn hơn: AI phát huy tốt nhất khi nằm trong một quy trình được thiết kế tốt — nó là một sự thay thế tồi cho việc không có quy trình nào cả.

### Ứng Dụng Vào Công Việc

- Thiết kế lại một thói quen học tập cá nhân của bản thân theo vòng lặp trigger–action–reward–investment — một streak hằng ngày, hoặc viết một bài ngắn chia sẻ công khai.
- Xây một script đánh giá prompt đơn giản với một tập test nhỏ, chạy prompt qua đó trước khi chốt dùng cho các dự án thực tế.
- Dựa vào tiêu chí nền tảng vững — không phải sự trôi chảy khi viết prompt — khi mentor hoặc đánh giá các bạn junior đang làm việc với công cụ AI.
- Thử áp dụng BMAD Method cho một dự án nhỏ, tách một workflow có AI hỗ trợ thành các vai trò Analyst/PM/Architect/Dev/QA riêng biệt thay vì một phiên chat dài không cấu trúc.

### Trải nghiệm trong event

Ngồi nghe **"FCAJ Community Day"** hoá ra lại ít nói về bản thân các công cụ AI, mà nói nhiều hơn về thói quen và quy trình của con người xung quanh chúng — điều này thành thật mà nói còn hữu ích hơn một buổi nói chuyện thuần về tính năng công cụ. Vài điều đọng lại với mình sau sự kiện:

**Việc học được nhìn lại như một thứ bạn thiết kế, chứ không phải thứ bạn tự ép mình vào kỷ luật.** Phần chia sẻ của Huynh Hoang Long đọng lại với mình nhiều nhất, chủ yếu vì sự so sánh có phần "khó chịu" của nó — chính vòng lặp khiến mạng xã hội khó dứt ra được lại là vòng lặp bạn có thể hướng vào việc học của chính mình, chỉ là đổi hướng sang một mục tiêu tốt hơn.

**Prompt engineering được đối xử như một bộ môn kỹ thuật thực thụ.** Nguyen Tuan Thinh đưa ra một lập luận khá thuyết phục rằng chất lượng prompt không nhất thiết phải sống trong vùng đoán mò — nó có thể được đo lường, kiểm thử và cải tiến giống hệt như bất kỳ artifact kỹ thuật nào khác.

**Một lời nhắc thực tế về ý nghĩa thật sự của "AI-ready" đối với sinh viên mới ra trường.** Phần chia sẻ của Khang là một cú "gõ đầu" hữu ích: kỹ năng viết prompt một mình nó sẽ không giúp một fresher nổi bật; nền tảng vững và khả năng đặt câu hỏi phản biện với output của AI mới là yếu tố quyết định.

**Một cách làm có cấu trúc thay thế cho việc chat AI vô tận.** Phần trình bày về **BMAD Method** của Thao gây ấn tượng tốt vì nó giải quyết trực tiếp một vấn đề mà nhiều người trong chúng ta từng âm thầm gặp phải — một cuộc trò chuyện AI dài, không có cấu trúc đơn giản là không trụ nổi khi dự án vượt qua một quy mô nhất định.

**Bài học rút ra nói chung:**
- Cả việc học lẫn việc viết prompt đều cải thiện ngay khi bạn bắt đầu coi chúng là những hệ thống cần thiết kế, thay vì những thứ chỉ cần cố gắng nhiều hơn.
- Công cụ AI khuếch đại bất kỳ quy trình nào nó được gắn vào — nên đầu tư vào quy trình mang lại lợi ích nhiều hơn là chạy theo một công cụ tốt hơn.
- "AI-ready" hoá ra là một tư duy và một bộ kỹ năng, không chỉ đơn thuần là quen tay với một cửa sổ chat.

#### Một số hình ảnh khi tham gia sự kiện

![FCAJ Community Day - Ảnh nhóm Event 1](/images/4-Event/4.1-Event1/picture1.jpg)

> Tổng thể, sự kiện đã kết nối thói quen học tập cá nhân, sự chặt chẽ trong prompt engineering, sự sẵn sàng cho sự nghiệp và phương pháp phát triển AI có cấu trúc thành một ý tưởng trung tâm: khai thác được giá trị thật từ AI không nằm nhiều ở bản thân công cụ, mà nằm ở hệ thống và kỷ luật bạn xây dựng xung quanh nó.
