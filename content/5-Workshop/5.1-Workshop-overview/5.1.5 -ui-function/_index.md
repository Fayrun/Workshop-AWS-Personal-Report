---
title : "UI Specification and System Functionality"
date : 2024-01-01 
weight : 5
chapter : false
pre : " <b> 5.1.5 </b> "
---

### 1. Login Interface

![image56.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/image56.png)

#### 1.1. Interface Overview
Functionality allowing existing users to perform identity authentication to access the **SmartDocAI** system. The system supports 2 primary authentication methods:
- **Native Authentication**: Using Email and Password.
- **Single Sign-On (SSO - Social Login)**: Using Google accounts via AWS Cognito Hosted UI (OAuth2 / OIDC).

#### 1.2. Layout Design and UI Components

| No. | Component Name | Component Type | Description / Display Spec |
| --- | --- | --- | --- |
| 1 | Form Title | `<h2>`, `<p>` | Displays title "Login" alongside "Register now!" link redirecting to `/register`. |
| 2 | Email | Input (Email) | Personal Email input field. Integrates Mail Icon on the left. |
| 3 | Password | Input (Password) | Password input field. Integrates Lock Icon on the left and show/hide password toggle (Eye/EyeOff Icon) on the right. |
| 4 | "Login" Button | Button | Primary Action Button. Displays spinning status (Spinner) "Logging in..." while request is processing. |
| 5 | Divider Line | Divider | Text "or continue with" separating the 2 login methods. |
| 6 | "Sign in with Google" Button | Button (Social) | Button featuring official Google SVG logo with shimmer hover effect. Clicking redirects to Cognito Hosted UI. |

#### 1.3. Input Data Validation Rules

| Data Field | Validation Rules | Displayed Error Message |
| --- | --- | --- |
| Email | - Cannot be left blank.<br/>- Must match standard email format (user@domain.com). | - "Please enter email."<br/>- "Invalid email format." |
| Password | Cannot be left blank. | "Please enter password." |

#### 1.4. Exception Handling & System Security

**Automatic Username Resolution:**
When a user creates an initial account via Google OAuth, Cognito generates an internal `username` like `Google_10293847...`.
When attempting manual login with Email/Password (after setting a native password), Cognito throws `UserNotFoundException` because internal username is not an email address.
The system automatically catches `UserNotFoundException` and calls Redux Thunk API `/api/auth/resolve-login-username` to look up the real username associated with that email and retries automatically without requiring user re-entry.

**Secure Session & Token Storage:**
JWT Tokens and session details (`id_token`, `access_token`, `refresh_token`, `auth_user`) are stored in `sessionStorage`, auto-clearing upon browser tab closure to mitigate security risks on shared devices.

**Status Feedback & Spam Form Prevention:**
The "Login" button immediately switches to `disabled` status with an active Spinner during AWS Cognito data requests, preventing double-clicking or duplicate request spam.

---

### 2. Registration Interface

Designed as a 2-step flow on a single screen:
- **Step 1 (Register)**: Input personal details and password.

![image57.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/image57.png)

- **Step 2 (Confirm OTP)**: Enter 6-digit verification code sent via Email to activate account.

![image58.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/image58.png)

#### 2.1. Interface Overview
The Account Registration interface (`/register`) serves as the entry point for new users into the **SmartDocAI** system, designed with a modern, intuitive Dark Mode interface following UX/UI standards.

#### 2.2. Layout Design and UI Components

##### 1. Step 1 Interface: Register Information Input

| No. | Component Name | Component Type | Description / Display Spec |
| --- | --- | --- | --- |
| 1 | Form Title | `<h2>`, `<p>` | Displays title "Create account" alongside "Sign in" link if account already exists. |
| 2 | Full Name | Input (Text) | User's full name input. Integrates User Icon. |
| 3 | Email | Input (Email) | Personal Email input (used as Cognito authentication Username). Integrates Mail Icon. |
| 4 | Phone | Input (Tel) | Contact phone number. Integrates Phone Icon. Displayed on same line as DOB on Desktop screens. |
| 5 | Date of Birth | Input (Date) | Date of birth selector. Integrates Calendar Icon. |
| 6 | Password | Input (Password) | Password input. Integrates Lock Icon and show/hide toggle (Eye/EyeOff Icon). |
| 7 | Confirm Password | Input (Password) | Re-enter password for match verification. Integrates show/hide toggle. |
| 8 | "Create account" Button | Button | Primary Action Button. Displays Loading Spinner when submitting request. |

##### 2. Step 2 Interface: OTP Code Verification

| No. | Component Name | Component Type | Description / Display Spec |
| --- | --- | --- | --- |
| 1 | Email Notification | `<p>` | Displays registered Email (highlighted in Blue) reminding user to check inbox. |
| 2 | Verification Code (OTP) | Input (Text) | 6-digit OTP input box, centered characters, prominent font formatting (`tracking-widest`). |
| 3 | "Activate account" Button | Button | Submits OTP code to Cognito/Backend API to finalize verification. |
| 4 | "Back to registration" Button | Button (Text) | Allows returning to Step 1 if user wants to modify details or Email. |

#### 2.3. Input Data Validation Rules

| Data Field | Validation Rules | Displayed Error Message |
| --- | --- | --- |
| Full Name | Cannot be left blank or whitespace only. | "Please enter full name." |
| Email | - Cannot be left blank.<br/>- Must match valid Email format (name@domain.com). | - "Please enter email."<br/>- "Invalid email format." |
| Phone Number | - Cannot be left blank.<br/>- Must contain between 9 and 11 digits. | - "Please enter phone number."<br/>- "Invalid phone number." |
| Date of Birth | - Cannot be left blank.<br/>- Birth year must be between 1900 and Current Year.<br/>- Cannot select future date. | - "Please select date of birth."<br/>- "Birth year must be between 1900 and [Current Year]."<br/>- "Date of birth cannot be in the future." |
| Password | - Cannot be left blank.<br/>- Minimum length 6 characters. | - "Please enter password."<br/>- "Password must be at least 6 characters." |
| Confirm Password | Must match Password field value exactly. | "Passwords do not match." |
| Verification Code (OTP) | Cannot be blank when clicking Activate. | "Please enter verification code." |

#### 2.4. Special Cases & Data Safety

**Account Linking Mechanism (PreSignUp Lambda):** When a user registered natively via Email/Password previously signs in with Google OAuth, the PreSignUp Trigger proactively links Google Provider to existing User ID instead of creating a new user entity. This prevents account fragmentation and data loss on Amazon S3 / DynamoDB.

**Automatic Profile Initialization (DynamoDB `ensure_user_profile`):** When accessing via Google for the first time, backend API automatically initializes standard profile record with `user_id` set to `sub` from Cognito JWT and `password_set = false`.

**Debounce & Loading State:** Action buttons switch to `disabled` status with a Spinning Loader during API request execution, eliminating duplicate request risks.

---

### 3. Main Dashboard Interface

![image59.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/image59.png)

#### 3.1. Interface Overview
Main Workspace (`/app`) is the control hub of the **SmartDocAI** system integrating document upload, chunking parameter setup, RAG strategy customization (Standard, Hybrid, Self-RAG, Co-RAG), AI Q&A, and citation viewing.
- **App Route**: `/app` (Requires authentication via `ProtectedRoute`).
- **Design Style**: Dark/Light Mode, 2-Column Responsive Layout optimized for parallel document lookup and AI communication.
- **UX Optimization**: Supports mobile devices via slide-over drawer sidebar (Mobile Drawer Overlay) opened via Hamburger Menu.

#### 3.2. Layout Design and UI Components
Main Dashboard layout is divided into 2 main functional areas: **Navigation & Document Management Sidebar (Sidebar - Left)** and **AI Chat & Q&A Workspace (Chat Area - Right)**.

#### 3.3. Main Components of Dashboard Interface

##### 3.3.1. Navigation & Document Management Sidebar (SmartSidebar)

- **Header and Logo Interface**:

![rId68.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId68.png)

  - *Functional description*: Displays brand name "SmartDocAI", Sidebar collapse toggle, Light/Dark theme toggle button.

- **System Status Interface**:

![rId69.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId69.png)

  - *Functional description*: Displays green/red indicator light showing connection status to AI service (Cloud LLM / Bedrock).

- **KPI Statistics Interface**:

  - *Before file upload*:

![rId70.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId70.png)

  - *After file upload*:

![rId71.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId71.png)

  - *Functional description*: Cards displaying overview metrics: Uploaded file count, Total pages (`totalPages`), Total text chunks (`totalChunks`).

- **Chunking Configuration Interface**:

![rId72.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId72.png)

  - *Functional description*: Slider control: Chunk Size (text chunk size, default 500-1000 tokens) and Chunk Overlap (overlap size, default 100-200 tokens).

- **File Upload Area Interface**:

  - *Before file upload*:

![rId73.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId73.png)

  - *After file upload*:

![rId74.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId74.png)

  - *Functional description*: Drag & Drop zone or click to select files. Supports PDF, DOCX formats up to size limit.

- **Document List Interface**:

  - *Before document processing*:

![rId75.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId75.png)

  - *After document processing*:

![rId76.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId76.png)

  - *Functional description*: Displays list of successfully processed documents, along with format info, page count, and individual delete buttons.

- **Action Buttons Interface**:
  - *Functional description*: Clears total session conversation history and deletes all uploaded documents from the system.

- **Chat History List Interface**:

  - *Before user asks question*:

![rId78.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId78.png)

  - *After user asks question*:

![rId79.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId79.png)

  - *Functional description*: Lists past chat sessions, allowing session selection or deletion.

##### 3.3.2. AI Chat & Q&A Workspace (ChatArea)

- **Unused State Interface**:

![rId79.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId79.png)

  - *Functional description*: Displays when no active conversation exists. Guides 3-step usage: 1. Upload doc → 2. System processing → 3. Ask questions.

- **Active Q&A Interface**:

![rId80.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId80.png)

  - *Functional description*: Message stream container displaying user questions and AI answers (SmartDocAI).

- **Citations Viewer Interface**:

![rId81.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId81.png)

  - *Functional description*: Block displayed below AI answers highlighting reference sources (Filename, Page Number, and corresponding text snippet).

- **Advanced Search Settings Panel**:

![rId82.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId82.png)

  - *Functional description*: Expandable panel allowing configuration of RAG algorithms: File Filter, Hybrid Search, Re-ranking, Self-RAG, Co-RAG.

- **Question Input Bar Interface**:

![rId83.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId83.png)

  - *Functional description*: Multi-line auto-expanding textarea, Send button (SendIcon).

#### 3.3. Search Strategy Rules & Customization

The `SearchSettings` control panel allows users to toggle advanced RAG technologies per search scenario:

| Strategy / Feature | Mechanism | Optimal Use Case |
| --- | --- | --- |
| File Filter | Restricts Vector search space to one or more user-selected files. | Searching data in 1 specified report/contract without mixing with other documents. |
| Hybrid Search | Combines Vector Search (Bedrock Titan Embeddings V2 60%) and Keyword Search (BM25 Algorithm 40%). | When questions contain specialized terminology, identifiers, contract numbers, or exact keywords. |
| Re-ranking | Uses multilingual Cross-Encoder model (`cross-encoder/mmarco-mMiniLMv2-L12-H384-v1`) to re-rank top-K most relevant text chunks (optimized for Vietnamese processing). | Filters noise out, increasing precision of context passed to LLM prompt. |
| Self-RAG | 3-step AI self-reflection loop: Query Rewriting, Relevance Filtering, Answer Grading. | Complex questions requiring AI to rewrite query if initial search info is insufficient. |
| Co-RAG (Multi-Agent) | Multi-agent collaboration: Semantic Agent (Vector), Keyword Agent (BM25), Conceptual Agent (LLM Concept). Merges results via voting, union, or intersection strategies. | Demands deep multi-perspective knowledge synthesis across documents. |

#### 3.4. State Handling & Exceptions on Main Page
- **No Document Uploaded**: Chat area displays welcome screen `WelcomeHero` with step-by-step guidance. Advanced RAG toggles in `SearchSettings` automatically switch to `disabled` status.
- **Corrupted File Processing Error**: System pops up Toast error warning immediately, preserving current UI state.
- **Data Isolation & Security (Per-User Isolation)**: All documents and chat history are bound to `user_id` (Cognito `sub`). User A cannot view or query User B's documents on S3/Vector Index.

---

### 4. User Profile Settings Interface

#### 4.1. Interface Overview

- **User Info Modification Interface**:

![rId84.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId84.png)

- **Change Password Interface**:

![rId85.png](/images/5-Workshop/5.1-Workshop-overview/5.1.5-ui-function/rId85.png)

Profile Settings interface (`Profile Settings`) allows users to view and update personal profile details within **SmartDocAI**:
- **Design Style**: Follows Dark Mode standards with deep dark gray/purple-blue palette (Deep Dark Theme) and modern thin-bordered Card layout.
- **Tab Bar Navigation**:
  - **Personal Info Tab (Active)**: Interface for updating avatar image and profile fields.
  - **Security Tab**: Interface for security settings and account password changes.
- **Dual-Card Layout**:
  - **Left Card (Avatar)**: Displays user avatar (initial letter or uploaded image), supporting drag-and-drop or click to update (JPG, PNG, WEBP, max 5MB).
  - **Right Card (Personal Info / Security)**: Depending on active Tab, displays form managing personal fields (Full Name, Email, Phone, Date of Birth) or password update form.

#### 4.2. Layout Design and UI Components (UI Specification)

| No. | Component Name | Component Type | Description / Display Spec |
| --- | --- | --- | --- |
| 1 | Back Button | Button / Link | Text ← Back, located at top-left corner, used to navigate back to Chat / Dashboard |
| 2 | Header User Info | Avatar & Text | Displays circular avatar containing initial letter (N) alongside Full Name (nam) and Email (namnp6567@gmail.com) at top corner |
| 3 | Tab Bar | Navigation Tabs | Comprises 2 tabs: Personal Info (default active) and Security |
| 4 | Avatar Card Title | Heading 3 | Title text Avatar at top of left Card |
| 5 | Avatar Display Area | Circle Avatar | Green circular frame displaying user's initial (e.g. N) or successfully uploaded avatar image |
| 6 | Image Dropzone | Dropzone Container | Dashed border box with guidance text: "Drag & drop image here or click to select - JPG, PNG, WEBP - max 5MB" |
| 7 | Change Avatar Button | Secondary Button | Thin-bordered button with camera icon and text Change avatar, opening local file picker |
| 8 | Info Card Title | Heading 3 & Subtitle | Title Personal Info with subtitle "Update your account display information" at top of right Card (under Info Tab) |
| 9 | FULL NAME Field | Input Textbox | Label: FULL NAME, displays current full name value (e.g. nam), allowing 2 - 100 characters |
| 10 | EMAIL Field | Input Textbox | Label: EMAIL, displays current account email address (e.g. namnp6567@gmail.com) |
| 11 | PHONE NUMBER Field | Input Textbox | Label: PHONE NUMBER, displays phone number (e.g. +84973251414), auto-formatted in E.164 |
| 12 | DATE OF BIRTH Field | Datepicker Input | Label: DATE OF BIRTH, containing calendar icon displaying birthdate (e.g. 09/01/2001), supporting day/month/year selection |
| 13 | Save Changes Button | Primary Button | Prominent blue button with save disk icon and text Save changes, sending update request to Backend |
| 14 | Security Card Title | Heading 3 & Subtitle | Title Security with subtitle "Change your login password" at top of right Card (under Security Tab) |
| 15 | CURRENT PASSWORD Field | Input Password | Label: CURRENT PASSWORD, displays masked characters, integrating show/hide toggle (Eye/EyeOff Icon) on right |
| 16 | NEW PASSWORD Field | Input Password | Label: NEW PASSWORD, placeholder "Minimum 6 characters", integrating show/hide toggle (Eye/EyeOff Icon) |
| 17 | CONFIRM NEW PASSWORD Field | Input Password | Label: CONFIRM NEW PASSWORD, placeholder "Re-enter new password", integrating show/hide toggle (Eye/EyeOff Icon) |
| 18 | Warning Note | Text | Displays note: "New password will apply to all subsequent login devices." |
| 19 | Update Password Button | Primary Button | Prominent blue button with save disk icon and text Update password to send password change request |

#### 4.3. Input Data Validation Rules Matrix

| Data Field | Validation Rules | Displayed Error Message |
| --- | --- | --- |
| Full Name | Required, cannot be blank | "Full name cannot be blank" |
| Full Name | Length between 2 and 100 characters, no dangerous special characters or HTML/Script tags (XSS protection) | "User name must be 2-100 characters without special characters" |
| Email | Required, cannot be blank | "Email cannot be blank" |
| Email | Must comply with standard RFC 5322 Email format (user@domain.com) | "Invalid email" |
| Phone Number | Required, cannot be blank | "Phone number cannot be blank" |
| Phone Number | Accepts Vietnam format (0901234567) or international E.164 (+84901234567), length 10 - 15 digits | "Invalid phone number" |
| Date of Birth | Required, cannot be blank | "Date of birth cannot be blank" |
| Date of Birth | Format YYYY-MM-DD, birth year between 1900 and current year | "Date of birth must follow YYYY-MM-DD, year from 1900 to 2026" |
| Avatar Image | File format must be image: JPG, PNG, WEBP | "Unsupported image file format. Accepts JPG, PNG, WEBP only" |
| Avatar Image | Maximum file size must not exceed 5 MB | "Image too large (...MB), max 5MB" |
| Current Password | Required, cannot be blank (if account has set native password) | "Please enter current password" |
| New Password | Required, minimum length 6 characters | "New password must be at least 6 characters" |
| Confirm New Password | Required, must match New Password field value exactly | "Confirmation password does not match" |

#### 4.4. Special Cases & Data Safety
- **Flexible `password_set` Flag Handling**: Ensures users signing in via Google OAuth for the first time are not blocked by "Current password", permitting easy creation of a native password to login via both methods in the future.
- **Input Image Validation**: Automatically blocks non-image files or raw files larger than 5MB prior to compression.
- **Reactive UI Sync**: When avatar image or Full Name changes on Profile page, Redux Store updates global state immediately, syncing information across Sidebar and Header instantly without requiring page reload.
