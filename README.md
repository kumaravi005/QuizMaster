# 🎓 QuizMaster Lite — Real-Time Online Examination System

[![Java](https://img.shields.io/badge/Java-17%2B-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)](https://openjdk.org/)
[![JavaFX](https://img.shields.io/badge/JavaFX-21-2D3748?style=for-the-badge&logo=java&logoColor=white)](https://openjfx.io/)
[![Javalin](https://img.shields.io/badge/Javalin-6.3.0-00A98F?style=for-the-badge)](https://javalin.io/)
[![SQLite](https://img.shields.io/badge/SQLite-3-003B57?style=for-the-badge&logo=sqlite&logoColor=white)](https://www.sqlite.org/)
[![Maven](https://img.shields.io/badge/Maven-3.9-C71A36?style=for-the-badge&logo=apache-maven&logoColor=white)](https://maven.apache.org/)
[![JUnit5](https://img.shields.io/badge/JUnit-5.11-25A162?style=for-the-badge&logo=junit5&logoColor=white)](https://junit.org/junit5/)

> A full-featured client–server online examination platform built with **JavaFX** and a high-performance **Javalin + SQLite** backend. Features server-enforced timers, real-time WebSocket live monitoring for faculty smartphones, crash-proof dual-layer auto-saving, and automated grading.
> 
> *Developed as an end-to-end coursework project for **Advanced Java Programming (CSE 307/309), SRM University AP**.*

---

## 📌 Table of Contents
1. [System Architecture](#-system-architecture)
2. [Key Features](#-key-features)
3. [Syllabus & Core Java Concepts](#-syllabus--core-java-concepts)
4. [Security & Reliability](#-security--reliability)
5. [Getting Started & Running](#-getting-started--running)
6. [Multi-Device Demo Guide](#-multi-device-demo-guide)
7. [API Specification](#-api-specification)
8. [Database Schema](#-database-schema)
9. [Automated Testing](#-automated-testing)
10. [Project Structure](#-project-structure)
11. [Troubleshooting](#-troubleshooting)

---

## 🏗 System Architecture

The project builds both the desktop client and the server from a single unified Maven codebase:

```
┌────────────────────────────────────────────────────────┐
│               JavaFX Desktop Application               │
│  (Student Exam Portal & Admin Management Dashboard)    │
│                                                        │
│  • UI Thread: JavaFX Application Thread                │
│  • QuizTimer (Runnable, nanoTime countdown)            │
│  • AnswerQueue (Producer-Consumer with wait/notify)    │
│  • AutoSaveThread (Daemon thread, Priority: MIN)       │
│  • Local Binary Storage: ~/.quizmaster/autosave/*.dat  │
└───────────────┬────────────────────────▲───────────────┘
                │                        │
         HTTP REST (JSON)         WebSocket Live Push
         (ApiClient)              (LiveClient)
                │                        │
                ▼                        │
┌────────────────────────────────────────┴───────────────┐
│              Javalin 6 Web & WebSocket Server          │
│                                                        │
│  • Auth / SessionFilter (PBKDF2 + Bearer Tokens)       │
│  • Controllers (Admin, Student, Auth, Live)            │
│  • Services (AttemptService, QuizService, AuthService) │
│  • DAOs (Repository<T> pattern, PreparedStatement)     │
│  • LiveHub (Concurrent broadcast to all listeners)     │
└───────────────┬────────────────────────┬───────────────┘
                │                        │
        JDBC Connection (ACID)    Static Web Files
                │                        │
                ▼                        ▼
     ┌─────────────────────┐   ┌─────────────────────────┐
     │ SQLite Database     │   │ Faculty Live Web Portal │
     │ (data/quiz.db)      │   │ (/live.html on mobile)  │
     └─────────────────────┘   └─────────────────────────┘
```

---

## ✨ Key Features

### 👨‍💼 Admin Capabilities
- **Quiz & Question Authoring:** Create, edit, and delete quizzes with customized time limits (1–180 minutes) and 4-option multiple choice questions.
- **Publish Workflow:** Quizzes can only be published when they contain at least one question. Students can only view and take published quizzes.
- **Real-Time Live Monitoring:** View students currently taking a quiz, their live progress (e.g., `6/10 answered`), and finished scores updating in real time.
- **Analytics & Visual Charts:** Interactive `BarChart` displaying student score distribution across ranges (`0-40%`, `40-60%`, `60-80%`, `80-100%`) and `PieChart` showing Pass vs. Fail ratios.
- **CSV Grade Export:** One-click export of complete student attempt records (rank, student name, score, total, percentage, time taken, timestamp) to standard CSV format.

### 🧑‍🎓 Student Experience
- **Timed Exam Room:** Clean, distraction-free interface with a countdown timer that highlights red under 60 seconds and auto-submits precisely at `00:00`.
- **Question Shuffling:** Questions are dynamically shuffled using the `attemptId` as the random seed, ensuring each student gets a randomized sequence while guaranteeing the exact same order upon resuming.
- **Crash Recovery & Dual-Layer Auto-Save:** Every answer selected is queued in the background (`AnswerQueue`) and simultaneously dispatched to the server and persisted to a local binary backup file (`DataOutputStream`). If the network disconnects or the app crashes, the student can reopen the quiz and resume with remaining time and saved answers intact.
- **Instant Animated Results:** Immediate server-side grading upon submission, featuring synchronized `FadeTransition` and `ScaleTransition` animations.
- **In-Depth Answer Review:** Detailed breakdown of all questions highlighting correct answers in green and student errors in red.
- **Personal Attempt History:** Track personal performance, scores, and review previous quiz submissions.

### 📱 Faculty Live Page (`/live.html`)
- Zero-install, responsive web page optimized for faculty smartphones and projector screens.
- Connects directly to Javalin via WebSockets (`/ws/live?quiz=<id>`) and automatically updates when any student starts, answers, or submits an exam.
- Includes automatic 3-second reconnection logic if Wi-Fi drops.

### ⌨️ Keyboard Shortcuts & AWT Help
- **`1`, `2`, `3`, `4`:** Select Options A, B, C, or D.
- **`←` / `→`:** Previous / Next question.
- **`F1`:** Launches the classic Java AWT Help Window (`Frame`, `WindowAdapter`, `KeyListener`, `ActionListener`).

---

## 📚 Syllabus & Core Java Concepts

This project covers all 5 units of the Advanced Java Programming syllabus:

| Unit | Syllabus Concept | Code Location & Implementation |
|---|---|---|
| **Unit 1: OOP & Inheritance** | Abstract classes & inheritance | [`User.java`](file:///src/main/java/com/quizapp/model/User.java) extended by [`Admin.java`](file:///src/main/java/com/quizapp/model/Admin.java) and [`Student.java`](file:///src/main/java/com/quizapp/model/Student.java). |
| | Interfaces & default methods | [`Gradable.java`](file:///src/main/java/com/quizapp/model/Gradable.java) interface provides `default double getPercentage()`. |
| | Immutability & `final` | [`PasswordUtil.java`](file:///src/main/java/com/quizapp/server/PasswordUtil.java) declared as `final class` with static methods. |
| **Unit 2: Polymorphism & Exceptions** | Dynamic method dispatch | Base [`QuizAppException.java`](file:///src/main/java/com/quizapp/exception/QuizAppException.java) hierarchy; HTTP handler calls `e.getHttpStatus()`. |
| | Overloading & Overriding | Method overloading in [`ApiClient.java`](file:///src/main/java/com/quizapp/client/ApiClient.java); method overriding in `Admin` / `Student`. |
| | Transactions & nested `try` | [`AttemptDAO.java`](file:///src/main/java/com/quizapp/server/dao/AttemptDAO.java) uses nested `try-catch-finally` with `commit()` and `rollback()`. |
| **Unit 3: Multithreading, I/O & Events** | Producer–Consumer pattern | [`AnswerQueue.java`](file:///src/main/java/com/quizapp/client/thread/AnswerQueue.java) uses `synchronized`, `wait()`, and `notify()`. |
| | Threads (`Runnable` vs `Thread`) | [`QuizTimer.java`](file:///src/main/java/com/quizapp/client/thread/QuizTimer.java) (`Runnable`) and [`AutoSaveThread.java`](file:///src/main/java/com/quizapp/client/thread/AutoSaveThread.java) (`Thread`). |
| | Binary Stream File I/O | `DataOutputStream` / `DataInputStream` in [`AutoSaveThread.java`](file:///src/main/java/com/quizapp/client/thread/AutoSaveThread.java) for local disk recovery. |
| | AWT Event Delegation Model | [`HelpWindow.java`](file:///src/main/java/com/quizapp/client/awt/HelpWindow.java) implements `ActionListener`, `KeyListener`, and `WindowAdapter`. |
| **Unit 4: JDBC & Collections** | Plain JDBC & Security | [`DatabaseManager.java`](file:///src/main/java/com/quizapp/server/DatabaseManager.java) and DAOs use `PreparedStatement` to prevent SQL injection. |
| | Advanced Collections | `ConcurrentHashMap` in [`SessionStore.java`](file:///src/main/java/com/quizapp/server/SessionStore.java) and [`LiveHub.java`](file:///src/main/java/com/quizapp/server/web/LiveHub.java); `Vector` in [`LiveResultsView.java`](file:///src/main/java/com/quizapp/client/ui/LiveResultsView.java). |
| | Deterministic Shuffle | `Collections.shuffle(list, new Random(attemptId))` in [`AttemptService.java`](file:///src/main/java/com/quizapp/server/service/AttemptService.java). |
| **Unit 5: Generics & JavaFX** | Generic interfaces & classes | Generic [`Repository<T>`](file:///src/main/java/com/quizapp/server/dao/Repository.java) and [`Pair<K, V>`](file:///src/main/java/com/quizapp/client/util/Pair.java). |
| | Bounded wildcards | `List<? extends Gradable>` in [`ScoreStats.java`](file:///src/main/java/com/quizapp/client/util/ScoreStats.java) and [`ResultExporter.java`](file:///src/main/java/com/quizapp/client/util/ResultExporter.java). |
| | Rich UI & Visual Analytics | `TableView`, `ListView`, `BarChart`, `PieChart`, `DropShadow`, and `ParallelTransition`. |

---

## 🔒 Security & Reliability

1. **Zero-Trust Client Architecture:**
   - Correct answers are **never sent to student clients**. Questions received during an attempt use [`QuestionView.java`](file:///src/main/java/com/quizapp/dto/QuestionView.java), which omits answer keys.
   - All grading is strictly executed on the server inside a single database transaction.
2. **Cryptographic Password Storage:**
   - Passwords hashed using industry-standard **PBKDF2WithHmacSHA256** with 210,000 iterations and a unique 16-byte cryptographically secure random salt per user.
   - Password comparisons use constant-time verification to prevent timing attacks.
3. **Anti-Cheat Time Enforcement:**
   - The server enforces the exam timer independently. Submissions arriving more than 30 seconds after deadline expiration (`GRACE_SECONDS = 30`) are flagged as late; answers sent after the deadline are discarded and only in-time auto-saves are graded.
4. **Idempotent Atomic Submissions:**
   - The client protects against double-clicks using an `AtomicBoolean.compareAndSet(false, true)`.
   - The server submission handler is idempotent: submitting an already-graded attempt safely returns the recorded result without re-grading.

---

## 🚀 Getting Started & Running

### Prerequisites
- **Java Development Kit (JDK) 17 or higher** (`java -version`).
- No separate Maven installation is required; the included Maven Wrapper (`mvnw` / `mvnw.cmd`) handles all dependencies.

### Step 1: Build the Project
Open a terminal in the project root:

**Windows (PowerShell):**
```powershell
.\mvnw.cmd clean package
```

**Linux / macOS:**
```bash
./mvnw clean package
```
*Wait until the output displays `BUILD SUCCESS`.*

---

### Step 2: Start the Central Server
Open Terminal 1 and run the compiled server JAR:

```bash
java -jar target/quizmaster-server.jar
```

When ready, the server logs:
```
QuizMaster server running on port 7070 (dev mode)
On this network: app -Dserver.url=http://192.168.1.15:7070   live page: http://192.168.1.15:7070/live.html
```

---

### Step 3: Launch the JavaFX Application
Open Terminal 2 and launch the client GUI:

**Windows (PowerShell):**
```powershell
.\mvnw.cmd javafx:run
```

**Linux / macOS:**
```bash
./mvnw javafx:run
```

*(To run multiple clients concurrently on the same machine, simply open another terminal and run the same command).*

---

### Step 4: Login Credentials & Testing
| Account Type | Username | Password | Notes |
|---|---|---|---|
| **Administrator** | `admin` | `admin123` | Pre-seeded default admin account |
| **Student** | *Custom* | *Custom* | Click **"New student? Create an account"** on the login screen |

- A sample quiz **"Java Basics"** (5 questions, 5 minutes) is pre-loaded and ready for testing.
- The live leaderboard can be accessed immediately in any browser at `http://localhost:7070/live.html`.

---

## 📡 Multi-Device Demo Guide (Same Local Wi-Fi)

You can demonstrate QuizMaster across multiple devices on the same Wi-Fi network without an active internet connection:

1. **Start the server on your laptop.** Note the LAN IP printed in the terminal (e.g. `http://192.168.1.50:7070`).
2. **Faculty / Evaluator:** Open `http://192.168.1.50:7070/live.html` on any smartphone or tablet browser.
3. **Classmates / Other Laptops:** Run the JavaFX app pointing to your host IP:
   ```powershell
   .\mvnw.cmd javafx:run "-Dserver.url=http://192.168.1.50:7070"
   ```
4. Watch the smartphone leaderboard and live question progress update instantaneously over WebSockets as classmates select answers and submit!

---

## 🔌 API Specification

All API endpoints exchange JSON and require `Authorization: Bearer <token>` unless marked public:

| Method | Endpoint | Access | Description |
|---|---|---|---|
| `GET` | `/health` | Public | Server health check (`{"status":"UP"}`) |
| `POST` | `/api/auth/register` | Public | Register student (`{fullName, username, password}`) |
| `POST` | `/api/auth/login` | Public | Authenticate user and receive bearer token |
| `GET` | `/api/live` | Public | List published quizzes available for live view |
| `GET` | `/api/live/{quizId}` | Public | Get snapshot `LiveState` for a quiz |
| `WS` | `/ws/live?quiz={id}` | Public | WebSocket stream pushing real-time `LiveState` |
| `GET` | `/api/admin/quizzes` | Admin | List all quizzes with question counts |
| `POST` | `/api/admin/quizzes` | Admin | Create a new quiz |
| `PUT / DELETE` | `/api/admin/quizzes/{id}` | Admin | Update quiz details or delete quiz (cascades) |
| `POST` | `/api/admin/quizzes/{id}/publish` | Admin | Publish quiz to students |
| `POST` | `/api/admin/quizzes/{id}/questions` | Admin | Add a question with options & answer key |
| `GET` | `/api/admin/quizzes/{id}/results` | Admin | Get all submitted attempts (used for CSV export) |
| `GET` | `/api/quizzes` | Student | List all published quizzes |
| `POST` | `/api/quizzes/{id}/start` | Student | Start or resume attempt (returns shuffled questions without keys) |
| `PUT` | `/api/attempts/{id}/progress` | Student | Auto-save progress (`{answers: {"1":"A"}}`) |
| `POST` | `/api/attempts/{id}/submit` | Student | Submit exam and trigger server grading |
| `GET` | `/api/attempts/{id}/review` | Student | Retrieve submitted attempt review with correct answers |
| `GET` | `/api/me/attempts` | Student | Retrieve own attempt history |

---

## 🗄 Database Schema

SQLite relational database with foreign key constraints enabled:

```
┌──────────────────┐          ┌───────────────────┐          ┌─────────────────────┐
│      users       │ 1      * │      quizzes      │ 1      * │      questions      │
├──────────────────┤──────────├───────────────────┤──────────├─────────────────────┤
│ id (PK)          │          │ id (PK)           │          │ id (PK)             │
│ username (UQ)    │          │ title             │          │ quiz_id (FK)        │
│ password_hash    │          │ subject           │          │ question_text       │
│ salt             │          │ time_limit_min    │          │ option_a .. d       │
│ full_name        │          │ is_published      │          │ correct_option      │
│ role             │          │ created_by (FK)   │          └─────────────────────┘
└────────┬─────────┘          └─────────┬─────────┘
         │ 1                            │ 1
         │                              │
         │ *                            │ *
┌────────┴──────────────────────────────┴─────────┐
│                    attempts                     │
├─────────────────────────────────────────────────┤
│ id (PK)                                         │
│ user_id (FK)                                    │
│ quiz_id (FK)                                    │
│ status ('IN_PROGRESS' / 'SUBMITTED')            │
│ started_at, submitted_at                        │
│ score, total, correct_count, wrong_count        │
│ unanswered_count, time_taken_sec, auto_submitted│
└────────────────────────┬────────────────────────┘
                         │ 1
                         │ *
            ┌────────────┴─────────────┐
            │     attempt_answers      │
            ├──────────────────────────┤
            │ attempt_id (PK, FK)      │
            │ question_id (PK, FK)     │
            │ chosen_option ('A'..'D') │
            └──────────────────────────┘
```

---

## 🧪 Automated Testing

The project includes **44 automated unit and integration tests** built on JUnit 5:

```bash
.\mvnw.cmd test
```

### Test Coverage Highlights:
- **`PasswordUtilTest`**: PBKDF2 hashing, unique salt generation, and verification correctness.
- **`AuthServiceTest`**: User registration, duplicate username rejection, authentication, and session tokens.
- **`QuizServiceTest`**: Quiz creation, question validation, and publication restrictions.
- **`AttemptServiceTest`**: Deterministic question shuffling, time limits, auto-save validation, late submission penalties, and grading arithmetic.
- **`LiveServiceTest`**: LiveState calculations, active writer counts, and leaderboard sorting.
- **`AnswerQueueTest`**: Multi-threaded producer–consumer testing of `put()` and `take()` with `wait()` and `notify()`.
- **`ScoreStatsTest`**: Distribution grouping and percentage calculations for graphical analytics.
- **`ResultExporterTest`**: CSV generation and character escaping.

---

## 📁 Project Structure

```
QuizMaster/
├── pom.xml                                  # Maven project descriptor & dependencies
├── mvnw / mvnw.cmd                          # Maven Wrapper scripts
├── Dockerfile                               # Containerization configuration
├── README.md                                # Project documentation
├── docs/                                    # In-depth architectural & viva documentation
│   ├── 01-overview.md                       # Functional & non-functional requirements
│   ├── 02-architecture.md                   # System design, thread models, and flowcharts
│   ├── 03-database.md                       # SQLite schema & key queries
│   ├── 04-ui-screens.md                     # Screen-by-screen breakdown
│   ├── 05-syllabus-mapping.md               # Code pointers for all 5 Java syllabus units
│   ├── 06-roadmap.md                        # Project milestone progression
│   ├── 07-setup.md                          # Detailed installation & configuration
│   ├── 08-api.md                            # Comprehensive REST & WebSocket API guide
│   ├── 09-deployment.md                     # Cloud & Docker deployment instructions
│   └── 10-viva-guide.md                     # 1-minute pitch & viva voce questions/answers
├── src/
│   ├── main/
│   │   ├── java/com/quizapp/
│   │   │   ├── model/                       # Domain models (User, Admin, Student, Quiz, Question)
│   │   │   ├── dto/                         # Data transfer objects & immutable records
│   │   │   ├── exception/                   # Strongly-typed HTTP exception hierarchy
│   │   │   ├── server/                      # ServerMain, DatabaseManager, PasswordUtil, SessionStore
│   │   │   │   ├── dao/                     # Generic Repository<T> & JDBC implementations
│   │   │   │   ├── service/                 # Business logic (Auth, Quiz, Attempt, Live)
│   │   │   │   └── web/                     # Javalin HTTP Controllers & WebSocket LiveHub
│   │   │   └── client/                      # JavaFX client application (App, Launcher, ApiClient)
│   │   │       ├── awt/                     # HelpWindow (AWT event delegation showcase)
│   │   │       ├── thread/                  # QuizTimer, AnswerQueue, AutoSaveThread
│   │   │       ├── ui/                      # Screen views (Login, Dashboard, Quiz, Stats, etc.)
│   │   │       └── util/                    # Async, AlertUtil, ResultExporter, ScoreStats
│   │   └── resources/
│   │       ├── schema.sql                   # SQLite schema initialization script
│   │       ├── client/styles.css            # JavaFX modern visual styling
│   │       └── public/                      # Static web assets (index.html, live.html)
│   └── test/java/com/quizapp/               # Automated JUnit 5 test suite (44 tests)
```

---

## ❓ Troubleshooting

| Issue | Cause | Solution |
|---|---|---|
| **"Cannot reach the server"** | Client opened before server started | Start the server first (`java -jar target\quizmaster-server.jar`), then launch the client. |
| **`Address already in use: bind`** | Port 7070 is already occupied by a previous server instance | Terminate any running Java server process or specify `PORT=8080 java -jar ...`. |
| **`'mvnw.cmd' is not recognized`** | Running command without local path prefix | Ensure you prefix with `.\` on PowerShell: `.\mvnw.cmd clean package`. |
| **Mobile devices cannot access `/live.html`** | Windows Firewall blocking inbound port 7070 | Allow Java through Windows Defender Firewall for Private Networks. |
| **Need a clean slate / reset DB** | Database has accumulated test data | Stop the server and delete the local `data/` folder. The server will recreate fresh tables and seed data upon restart. |

---

## 📄 License
This project is licensed under the MIT License — see the LICENSE file for details.
