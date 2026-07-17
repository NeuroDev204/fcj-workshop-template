# Internship Info Updates Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Update internship dates to 17/4/2026 - 30/7/2026 (approx. 15 weeks) across all relevant pages, and add the CloudFront production application link to the Workshop section.

**Architecture:** Modifying markdown files inside `content/` folder in the Hugo repository.

**Tech Stack:** Hugo Markdown

## Global Constraints

- Vietnamese date format: DD/MM/YYYY (e.g., 17/04/2026, 30/07/2026).
- English date format: Month DD, YYYY (e.g., April 17, 2026, July 30, 2026).
- CloudFront link: `https://d1di1pzmfypszp.cloudfront.net/`.

---

### Task 1: Update Dates in Root / Home Pages

**Files:**
- Modify: `content/_index.vi.md:25-27`
- Modify: `content/_index.md:28-30`

- [ ] **Step 1: Modify content/_index.vi.md**
  Update line 26 from:
  ```markdown
  &emsp; **Thời gian thực tập:** Từ ngày 20/04/2026 đến ngày 12/7/2026
  ```
  to:
  ```markdown
  &emsp; **Thời gian thực tập:** Từ ngày 17/04/2026 đến ngày 30/07/2026
  ```

- [ ] **Step 2: Modify content/_index.md**
  Update line 29 from:
  ```markdown
  &emsp; **Internship Duration:** From 20/04/2026 to 12/7/2026
  ```
  to:
  ```markdown
  &emsp; **Internship Duration:** From 17/04/2026 to 30/07/2026
  ```

- [ ] **Step 3: Commit Task 1**
  ```bash
  git add content/_index.vi.md content/_index.md
  git commit -m "feat: update internship dates in home pages"
  ```

---

### Task 2: Update Dates in Worklog & Self-Evaluation Pages

**Files:**
- Modify: `content/1-Worklog/_index.vi.md:8-10`
- Modify: `content/1-Worklog/_index.md:8-10`
- Modify: `content/6-Self-evaluation/_index.vi.md:8-10`
- Modify: `content/6-Self-evaluation/_index.md:8-10`

- [ ] **Step 1: Modify content/1-Worklog/_index.vi.md**
  Update line 9 from:
  ```markdown
  Worklog dưới đây ghi lại toàn bộ quá trình thực tập của mình tại chương trình **Workforce Bootcamp - First Cloud AI Journey** của AWS Việt Nam, kéo dài **12 tuần** (từ 20/04/2026 đến 12/07/2026, khoảng 3 tháng). 5 tuần đầu là giai đoạn tự học theo chương trình AWS Study Group với các dịch vụ nền tảng; 7 tuần còn lại là giai đoạn cùng team 5 người xây dựng project cuối kỳ **PeriodIQ**, trong đó mình phụ trách hạng mục **CI/CD & Monitoring** (AWS CodePipeline, CodeBuild, CloudFormation/SAM, CloudWatch).
  ```
  to:
  ```markdown
  Worklog dưới đây ghi lại toàn bộ quá trình thực tập của mình tại chương trình **Workforce Bootcamp - First Cloud AI Journey** của AWS Việt Nam, kéo dài **15 tuần** (từ 17/04/2026 đến 30/07/2026, khoảng 3.5 tháng). 5 tuần đầu là giai đoạn tự học theo chương trình AWS Study Group với các dịch vụ nền tảng; 7 tuần còn lại là giai đoạn cùng team 5 người xây dựng project cuối kỳ **PeriodIQ**, trong đó mình phụ trách hạng mục **CI/CD & Monitoring** (AWS CodePipeline, CodeBuild, CloudFormation/SAM, CloudWatch).
  ```

- [ ] **Step 2: Modify content/1-Worklog/_index.md**
  Update line 9 from:
  ```markdown
  This worklog covers my entire internship at the **Workforce Bootcamp - First Cloud AI Journey** program run by AWS Vietnam, spanning **12 weeks** (April 20, 2026 to July 12, 2026, about 3 months). The first 5 weeks were a self-paced AWS Study Group phase covering foundational services; the remaining 7 weeks were spent building the capstone project **PeriodIQ** with a 5-person team, where I owned the **CI/CD & Monitoring** track (AWS CodePipeline, CodeBuild, CloudFormation/SAM, CloudWatch).
  ```
  to:
  ```markdown
  This worklog covers my entire internship at the **Workforce Bootcamp - First Cloud AI Journey** program run by AWS Vietnam, spanning **15 weeks** (April 17, 2026 to July 30, 2026, about 3.5 months). The first 5 weeks were a self-paced AWS Study Group phase covering foundational services; the remaining 7 weeks were spent building the capstone project **PeriodIQ** with a 5-person team, where I owned the **CI/CD & Monitoring** track (AWS CodePipeline, CodeBuild, CloudFormation/SAM, CloudWatch).
  ```

- [ ] **Step 3: Modify content/6-Self-evaluation/_index.vi.md**
  Update line 9 from:
  ```markdown
  Trong suốt thời gian thực tập tại **Công ty TNHH Amazon Web Services Việt Nam** (chương trình **Workforce Bootcamp - First Cloud AI Journey**) từ **20/04/2026** đến **12/07/2026**, tôi đã có cơ hội học hỏi, rèn luyện và áp dụng kiến thức đã được trang bị tại trường vào môi trường làm việc thực tế.  
  ```
  to:
  ```markdown
  Trong suốt thời gian thực tập tại **Công ty TNHH Amazon Web Services Việt Nam** (chương trình **Workforce Bootcamp - First Cloud AI Journey**) từ **17/04/2026** đến **30/07/2026**, tôi đã có cơ hội học hỏi, rèn luyện và áp dụng kiến thức đã được trang bị tại trường vào môi trường làm việc thực tế.  
  ```

- [ ] **Step 4: Modify content/6-Self-evaluation/_index.md**
  Update line 9 from:
  ```markdown
  During my internship at **Amazon Web Services Viet Nam Company Limited** (the **Workforce Bootcamp - First Cloud AI Journey** program) from **April 20, 2026** to **July 12, 2026**, I had the opportunity to learn, practice, and apply the knowledge acquired in school to a real-world working environment.  
  ```
  to:
  ```markdown
  During my internship at **Amazon Web Services Viet Nam Company Limited** (the **Workforce Bootcamp - First Cloud AI Journey** program) from **April 17, 2026** to **July 30, 2026**, I had the opportunity to learn, practice, and apply the knowledge acquired in school to a real-world working environment.  
  ```

- [ ] **Step 5: Commit Task 2**
  ```bash
  git add content/1-Worklog/_index.vi.md content/1-Worklog/_index.md content/6-Self-evaluation/_index.vi.md content/6-Self-evaluation/_index.md
  git commit -m "feat: update internship dates in worklog and self-evaluation pages"
  ```

---

### Task 3: Add CloudFront Link in Workshop Section

**Files:**
- Modify: `content/5-Workshop/_index.vi.md`
- Modify: `content/5-Workshop/_index.md`

- [ ] **Step 1: Modify content/5-Workshop/_index.vi.md**
  Insert the Production Link section under the Overview (`#### Tổng quan`) header, around line 14:
  ```markdown
  #### Link Production
  
  * **Đường dẫn ứng dụng:** [https://d1di1pzmfypszp.cloudfront.net/](https://d1di1pzmfypszp.cloudfront.net/)
  ```

- [ ] **Step 2: Modify content/5-Workshop/_index.md**
  Insert the Production Link section under the Overview (`#### Overview`) header, around line 14:
  ```markdown
  #### Production Link
  
  * **Live Application:** [https://d1di1pzmfypszp.cloudfront.net/](https://d1di1pzmfypszp.cloudfront.net/)
  ```

- [ ] **Step 3: Commit Task 3**
  ```bash
  git add content/5-Workshop/_index.vi.md content/5-Workshop/_index.md
  git commit -m "feat: add production link in workshop section"
  ```

---

### Task 4: Verify and Build Site

- [ ] **Step 1: Run Hugo server / build to ensure no errors**
  Run: `hugo`
  Expected: Hugo builds site successfully to `public/` folder without errors.
