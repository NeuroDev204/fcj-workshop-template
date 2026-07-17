# Design Spec: Internship Info Updates

Updating the internship dates from `20/04/2026 - 12/7/2026` to `17/04/2026 - 30/7/2026` (approx. 15 weeks) across all relevant pages, and adding the CloudFront production application link to the Workshop section.

## Goals

1. Consistent date changes across English and Vietnamese versions of the report.
2. Prominent placement of the CloudFront production link in the Workshop section.

## Affected Files & Detailed Changes

### 1. Root / Home Pages

#### [content/_index.vi.md](file:///home/neuro/Documents/AWS%20LAB/fcj-workshop-template/content/_index.vi.md)
* **Target Content:** `&emsp; **Thời gian thực tập:** Từ ngày 20/04/2026 đến ngày 12/7/2026`
* **Replacement:** `&emsp; **Thời gian thực tập:** Từ ngày 17/04/2026 đến ngày 30/07/2026`

#### [content/_index.md](file:///home/neuro/Documents/AWS%20LAB/fcj-workshop-template/content/_index.md)
* **Target Content:** `&emsp; **Internship Duration:** From 20/04/2026 to 12/7/2026`
* **Replacement:** `&emsp; **Internship Duration:** From 17/04/2026 to 30/07/2026`

---

### 2. Worklog Section

#### [content/1-Worklog/_index.vi.md](file:///home/neuro/Documents/AWS%20LAB/fcj-workshop-template/content/1-Worklog/_index.vi.md)
* **Target Content:** `kéo dài **12 tuần** (từ 20/04/2026 đến 12/07/2026, khoảng 3 tháng).`
* **Replacement:** `kéo dài **15 tuần** (từ 17/04/2026 đến 30/07/2026, khoảng 3.5 tháng).`

#### [content/1-Worklog/_index.md](file:///home/neuro/Documents/AWS%20LAB/fcj-workshop-template/content/1-Worklog/_index.md)
* **Target Content:** `spanning **12 weeks** (April 20, 2026 to July 12, 2026, about 3 months).`
* **Replacement:** `spanning **15 weeks** (April 17, 2026 to July 30, 2026, about 3.5 months).`

---

### 3. Self-Assessment / Evaluation

#### [content/6-Self-evaluation/_index.vi.md](file:///home/neuro/Documents/AWS%20LAB/fcj-workshop-template/content/6-Self-evaluation/_index.vi.md)
* **Target Content:** `từ **20/04/2026** đến **12/07/2026**, tôi đã`
* **Replacement:** `từ **17/04/2026** đến **30/07/2026**, tôi đã`

#### [content/6-Self-evaluation/_index.md](file:///home/neuro/Documents/AWS%20LAB/fcj-workshop-template/content/6-Self-evaluation/_index.md)
* **Target Content:** `from **April 20, 2026** to **July 12, 2026**, I had`
* **Replacement:** `from **April 17, 2026** to **July 30, 2026**, I had`

---

### 4. Workshop Section (Adding Production Link)

#### [content/5-Workshop/_index.vi.md](file:///home/neuro/Documents/AWS%20LAB/fcj-workshop-template/content/5-Workshop/_index.vi.md)
* **Change:** Add a new section `#### Link Production` with the link `https://d1di1pzmfypszp.cloudfront.net/` right under the `#### Tổng quan` block.
* **Content:**
  ```markdown
  #### Link Production
  
  * **Đường dẫn ứng dụng:** [https://d1di1pzmfypszp.cloudfront.net/](https://d1di1pzmfypszp.cloudfront.net/)
  ```

#### [content/5-Workshop/_index.md](file:///home/neuro/Documents/AWS%20LAB/fcj-workshop-template/content/5-Workshop/_index.md)
* **Change:** Add a new section `#### Production Link` with the link `https://d1di1pzmfypszp.cloudfront.net/` right under the `#### Overview` block.
* **Content:**
  ```markdown
  #### Production Link
  
  * **Live Application:** [https://d1di1pzmfypszp.cloudfront.net/](https://d1di1pzmfypszp.cloudfront.net/)
  ```
