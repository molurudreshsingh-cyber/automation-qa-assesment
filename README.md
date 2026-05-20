# Technical Assessment: Automation & QA Developer

## ⚙️ Task 2: n8n Core Integration Workflow
* **APIs Used:** GitHub REST API v3 (`/search/repositories`).
* **Why Chosen:** It provides a reliable, live stream of real-world developer tools to simulate a business "morning brief" digest.
* **Data Transformation:** The inline JavaScript Code node evaluates the raw GitHub payload and runs a `.slice(0, 5)` array method. This limits the stream to the Top 5 most starred repositories, saving system processing memory and keeping the summary concise.
* **Error Mitigation:** The workflow nodes leverage explicit "Continue On Fail" routing behaviors. If GitHub hits a network timeout or a rate-limit error, the node captures the failure context dynamically instead of crashing the pipeline, allowing safe failover execution.

---

## 🚨 Task 1: Web App QA & Debug Report
Conducted boundary and exploratory validation against the hosted web ecosystem application to identify production-blocking bugs.

### Bug Tracking Table
| # | Title / Summary | Steps to Reproduce | Expected vs Actual | Severity | Suspected Cause |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | Client-side validation bypass on registration email field. | 1. Go to Register page.<br>2. Intercept request or bypass HTML5 validation to submit an invalid email format (e.g., `user@com`). | **Expected:** API/UI should reject with a clear 422 validation error.<br>**Actual:** Server accepts invalid email structures, creating corrupt profiles. | High | Missing robust back-end Regex validation on the user registration controller. |
| **2** | Article creation page lacks unsaved changes warning. | 1. Log in.<br>2. Click "New Article" and type a long draft.<br>3. Accidentally click the home logo or back button. | **Expected:** A browser prompt warning the user of unsaved changes.<br>**Actual:** App instantly navigates away, destroying all user progress. | Medium | Missing frontend router guard (`BeforeUnmount` / `canDeactivate`) on the editor route. |
| **3** | Exploding UI layout with massive continuous text strings. | 1. Create an article.<br>2. In the description, paste a 500-character string with no spaces (e.g., `AAAA...`).<br>3. View the article in the feed. | **Expected:** Text should break gracefully or truncate with ellipses.<br>**Actual:** Text overflows the containers, breaking the entire layout grid UI. | Medium | Missing CSS styling property `word-break: break-word;` or `overflow: hidden;` on feed cards. |
| **4** | Broken pagination state on browser back navigation. | 1. Navigate to page 4 of the global feed.<br>2. Click an article to view it.<br>3. Click the browser's back button. | **Expected:** Return to page 4 of the feed.<br>**Actual:** Resets entirely to page 1, forcing the user to re-scroll and find their place. | Low | Pagination state is held in ephemeral local component state instead of URL query parameters (`?page=4`). |
| **5** | Authorization token leak via unhandled error console logs. | 1. Open Browser DevTools (Console).<br>2. Intentionally trigger a failed profile update or action while logged in. | **Expected:** Silent fail or generic user-facing UI error toast.<br>**Actual:** The entire Axios/Fetch error object—including the user's plain JWT bearer token—is dumped into console logs. | High | Leftover development code (`console.log(error)`) in the global API error interceptor catch block. |

### Root-Cause Analysis (RCA) - Authorization Token Leak
* **Defect Behavior:** When an authenticated API operation encounters a system fault, the frontend leaks the user's active JSON Web Token (JWT) credentials in plaintext straight into the developer tools web log.
* **Underlying Trigger:** A production build contains an unhandled `console.log(error)` catch block within the central API client interceptor code instead of filtering server details out before publication.
* **Remediation Action:** Clean out development logging streams and wrap global client handlers in an environment conditional rule (`if (process.env.NODE_ENV === 'development')`) to isolate cryptographic tokens inside protected application memory.
