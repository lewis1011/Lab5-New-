# Offline Posts Manager - Flutter Lab 5 Report

## Student Information
**Course:** Mobile Computing  
**Lab Number:** Lab 5 - Local Storage in Flutter  
**Assignment Name:** Offline Posts Manager with SQLite  
**Date:** April 25, 2026  

**Student Details:**
- **Name:** Lewis Ngendahimana
- **Student ID:** 223026674
- **Campus:** Gako Campus
- **Submission Date:** 24/04/2026

---

## Project Overview

**Application Name:** Offline Posts Manager  
**Description:** A Flutter mobile application that enables staff members of a media company to manage posts locally without requiring internet connectivity. The app implements a complete CRUD (Create, Read, Update, Delete) system with SQLite database integration for persistent local storage.

**GitHub Repository:** https://github.com/lewis1011/Lab5-New-.git

---

## Lab Objectives

### Primary Goals
1. ✅ Practice local database integration using SQLite in Flutter
2. ✅ Implement complete CRUD operations for post management
3. ✅ Build a responsive and stateful UI that updates in real-time
4. ✅ Handle database exceptions and errors gracefully
5. ✅ Understand asynchronous database operations in Flutter

### Application Requirements
Staff members should be able to:
- [x] View all posts stored locally
- [x] Read the details of a post
- [x] Add a new post
- [x] Edit an existing post
- [x] Delete a post

---

## Application Features

### 1. Home Screen - View All Posts
- Displays a list of all stored posts in reverse chronological order (newest first)
- Shows post title and preview of content
- Displays creation and update timestamps
- Pull-to-refresh functionality to reload posts
- Empty state with helpful message when no posts exist
- Action buttons for edit and delete operations

![Home Screen with Posts List](screenshots/Screenshot%202026-04-25%20111648.png)

### 2. Add Post Screen
- Form with title and content input fields
- Real-time character count validation
- Input validation (title max 200 chars, content max 5000 chars)
- Error handling and user feedback
- Save and cancel buttons

### 3. Edit Post Screen
- Pre-populated form with existing post data
- Similar validation as add screen
- Displays last update timestamp
- Updates the post with new content
- Tracks update time automatically

### 4. Delete Post
- Confirmation dialog before deletion
- Permanent removal from local database
- Success notification

---

## Technical Implementation

### Dependencies Used and Rationale

#### 1. **sqflite** (Version ^2.3.0)
- **Purpose:** SQLite plugin for Flutter
- **Why It's Necessary:**
  - Provides reliable local data persistence
  - Works offline without internet connectivity
  - Suitable for structured data like posts
  - Lightweight and efficient for mobile devices
  - Cross-platform (iOS and Android)

#### 2. **path_provider** (Version ^2.1.1)
- **Purpose:** Provides access to device file system directories
- **Why It's Used:**
  - Gets application documents directory for safe database storage
  - Platform-specific path handling (iOS/Android)
  - Ensures database persists across app restarts

#### 3. **intl** (Version ^0.19.0)
- **Purpose:** Internationalization and localization library
- **Why It's Used:**
  - Formats dates and times consistently
  - Better readability of timestamps in UI

#### 4. **Flutter SDK** (built-in features)
- **Material Design:** Material UI components and design patterns
- **Async/Await:** For managing asynchronous database operations

---

## Database Implementation

### SQLite Architecture

#### Database vs Table
- **Database:** The entire `posts_manager.db` file stored on the device
  - Acts as container for all related tables
  - Persistent storage across app sessions
  - Single database file for this app

- **Table:** The `posts` table within the database
  - Schema: Contains columns for id, title, content, created_at, updated_at
  - Each row represents a single post
  - Automatically configured on first app launch

### Database Schema
```sql
CREATE TABLE posts (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  title TEXT NOT NULL,
  content TEXT NOT NULL,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL
)
```

**Column Definitions:**
- `id`: Auto-incrementing primary key (unique identifier)
- `title`: Post title (required, up to 200 characters)
- `content`: Post body (required, up to 5000 characters)
- `created_at`: ISO8601 timestamp of creation (immutable)
- `updated_at`: ISO8601 timestamp of last modification

### CRUD Operations Implementation

#### CREATE (INSERT)
```dart
Future<int> insertPost(Post post) async {
  final db = await database;
  return await db.insert(
    'posts',
    post.toMap(),
    conflictAlgorithm: ConflictAlgorithm.replace,
  );
}
```
- Converts Post object to Map
- Inserts into database with automatic ID generation
- Returns the newly created post's ID

#### READ (SELECT)
```dart
Future<List<Post>> getAllPosts() async {
  final db = await database;
  final List<Map<String, dynamic>> maps = await db.query(
    'posts',
    orderBy: 'updated_at DESC',
  );
  return List.generate(maps.length, (i) => Post.fromMap(maps[i]));
}
```
- Queries all posts ordered by most recently updated first
- Converts database rows to Post objects
- Returns empty list if no posts exist

#### UPDATE
```dart
Future<int> updatePost(Post post) async {
  final db = await database;
  return await db.update(
    'posts',
    post.toMap(),
    where: 'id = ?',
    whereArgs: [post.id],
  );
}
```
- Updates post with matching ID
- Overwrites all fields except ID
- Updated timestamp automatically set

#### DELETE
```dart
Future<int> deletePost(int id) async {
  final db = await database;
  return await db.delete(
    'posts',
    where: 'id = ?',
    whereArgs: [id],
  );
}
```
- Removes post with specified ID
- Returns number of rows deleted (1 if successful)

### Asynchronous Database Operations

**How Flutter Interacts with Database Asynchronously:**

1. **Future-Based API:** All database operations return `Future` objects
   ```dart
   Future<Database> get database async { ... }
   Future<int> insertPost(Post post) async { ... }
   ```

2. **Async/Await Pattern:** Used to wait for database operations
   ```dart
   final posts = await _databaseHelper.getAllPosts();
   setState(() { _posts = posts; });
   ```

3. **Non-Blocking Execution:** Database operations don't freeze the UI
   - Operations run on background isolates
   - UI remains responsive during database access
   - Users can interact with app while queries execute

4. **Error Handling:** Exceptions propagated through Future chain
   ```dart
   try {
     final posts = await _databaseHelper.getAllPosts();
   } catch (e) {
     // Handle error
   }
   ```

---

## Exception Handling

### Database Not Initialized
**Scenario:** Database fails to initialize on app launch
```dart
Future<Database> _initDatabase() async {
  try {
    final documentsDirectory = await getApplicationDocumentsDirectory();
    final path = join(documentsDirectory.path, 'posts_manager.db');
    return await openDatabase(path, version: 1, onCreate: _onCreate);
  } catch (e) {
    throw Exception('Failed to initialize database: $e');
  }
}
```
**Handling:** Try-catch block in initialization, error propagated to UI

### Insert/Update/Delete Errors
**Scenario:** Corrupted data or database constraints violated
```dart
try {
  await _databaseHelper.insertPost(post);
  ScaffoldMessenger.of(context).showSnackBar(
    const SnackBar(content: Text('Post created successfully')),
  );
} catch (e) {
  setState(() {
    _errorMessage = 'Error saving post: ${e.toString()}';
  });
}
```
**Handling:** 
- Validation before operations
- User-friendly error messages displayed in UI
- Snack bars notify users of successes/failures

### Invalid or Corrupted Data
**Scenario:** Received malformed data from database
```dart
factory Post.fromMap(Map<String, dynamic> map) {
  return Post(
    id: map['id'] as int?,
    title: map['title'] as String,
    content: map['content'] as String,
    createdAt: DateTime.parse(map['created_at'] as String),
    updatedAt: DateTime.parse(map['updated_at'] as String),
  );
}
```
**Handling:**
- Type casting ensures data consistency
- DateTime parsing validates date format
- Invalid data causes exception caught in UI layer

### Input Validation
```dart
bool _validateInput() {
  if (_titleController.text.isEmpty) {
    setState(() { _errorMessage = 'Please enter a title'; });
    return false;
  }
  if (_titleController.text.length > 200) {
    setState(() { _errorMessage = 'Title must be less than 200 characters'; });
    return false;
  }
  // ... more validations
  return true;
}
```

---

## Application Screenshots

### Screenshot 1: Home Screen with Posts
![Home Screen - Posts List](screenshots/Screenshot%202026-04-25%20111648.png)
**Description:** Main screen displaying all stored posts in reverse chronological order with edit/delete buttons visible on each post card.

---

### Screenshot 2: Empty Home Screen
![Empty Home Screen](screenshots/Screenshot%202026-04-25%20112048.png)
**Description:** Empty state with helpful message "No posts yet" and prompt to create the first post using the floating action button.

---

### Screenshot 3: Add Post Screen
![Add Post Screen](screenshots/Screenshot%202026-04-25%20112156.png)
**Description:** Form for creating a new post with title and content input fields, character count validation, and save/cancel buttons.

---

### Screenshot 4: Additional Screen
![App Screen](screenshots/Screenshot%202026-04-25%20112237.png)
**Description:** Shows additional app functionality or state.

---

## Project Structure

```
Lab5(New)/
├── lib/
│   ├── models/
│   │   └── post.dart              # Post model with toMap() and fromMap()
│   ├── services/
│   │   └── database_helper.dart   # SQLite database operations (singleton pattern)
│   ├── screens/
│   │   ├── home_screen.dart       # Main screen displaying posts list
│   │   ├── add_post_screen.dart   # Form for creating new posts
│   │   └── edit_post_screen.dart  # Form for editing existing posts
│   ├── widgets/
│   │   └── post_card.dart         # Reusable post card widget
│   └── main.dart                  # App entry point
├── pubspec.yaml                   # Dependencies and project configuration
├── README.md                       # This lab report
└── screenshots/                   # Directory for app screenshots
    ├── [screenshot_1.png]
    ├── [screenshot_2.png]
    └── ...
```

---

## How to Run the Application

### Prerequisites
- Flutter SDK installed (version 3.0.0+)
- Android emulator or iOS simulator
- A terminal/command prompt

### Setup Instructions
1. **Clone the repository:**
   ```bash
   git clone https://github.com/lewis1011/Lab5-New-.git
   cd Lab5-New
   ```

2. **Install dependencies:**
   ```bash
   flutter pub get
   ```

3. **Run the app:**
   ```bash
   flutter run
   ```

4. **Run on specific device:**
   ```bash
   flutter run -d emulator-id
   ```

### Testing CRUD Operations
1. **Create:** Tap the '+' button to add a new post
2. **Read:** View all posts on the home screen
3. **Update:** Tap edit icon on any post to modify it
4. **Delete:** Tap delete icon and confirm deletion

---

## Testing Results

### Test Case 1: Create Post
- **Input:** Title: "First Post", Content: "This is my first post"
- **Expected:** Post saved and appears in list
- **Result:** ✅ PASSED
- **Screenshot:** See Add Post Screen above

### Test Case 2: View Posts
- **Input:** Multiple posts saved in database
- **Expected:** All posts display in reverse chronological order
- **Result:** ✅ PASSED
- **Screenshot:** ![Home Screen - Posts List](screenshots/Screenshot%202026-04-25%20111648.png)

### Test Case 3: Edit Post
- **Input:** Modify existing post title and content
- **Expected:** Changes saved and reflected in list
- **Result:** ✅ PASSED
- **Screenshot:** See Add Post Screen functionality applied

### Test Case 4: Delete Post
- **Input:** Delete a post from the list
- **Expected:** Confirmation dialog appears, post removed after confirmation
- **Result:** ✅ PASSED
- **Screenshot:** Verified through home screen testing

### Test Case 5: Empty State
- **Input:** Delete all posts
- **Expected:** Empty screen with helpful message
- **Result:** ✅ PASSED
- **Screenshot:** ![Empty Home Screen](screenshots/Screenshot%202026-04-25%20112048.png)

### Test Case 6: Input Validation
- **Input:** Try to save post with empty title or content
- **Expected:** Error message displayed, post not saved
- **Result:** ✅ PASSED
- **Screenshot:** Validation messages shown in Add Post Screen

---

## Challenges and Solutions

### Challenge 1: Database Initialization
**Issue:** Database not initializing on first app launch
**Solution:** Implemented singleton pattern in DatabaseHelper with proper async initialization and try-catch error handling

### Challenge 2: UI Not Updating After Operations
**Issue:** Posts list not refreshing after add/edit/delete
**Solution:** Implemented callbacks and proper state management using setState() after each database operation

### Challenge 3: Date/Time Handling
**Issue:** DateTime conversion and formatting
**Solution:** Used ISO8601 format for storage and intl package for consistent formatting in UI

### Challenge 4: Exception Handling
**Issue:** Unhandled database exceptions crashing app
**Solution:** Wrapped all database operations in try-catch blocks with user-friendly error messages

---

## Key Learnings

### 1. SQLite for Mobile Development
- SQLite is lightweight and perfect for local storage needs
- No server required, data persists locally
- Excellent for offline-first applications

### 2. Asynchronous Programming in Flutter
- Database operations must be asynchronous to prevent UI blocking
- Future-based API allows non-blocking data access
- Proper async/await handling essential for good UX

### 3. State Management
- setState() updates UI after database operations
- Proper error handling prevents app crashes
- User feedback (SnackBars, AlertDialogs) improves UX

### 4. CRUD Pattern
- Implementation is straightforward with sqflite plugin
- Validation and error handling are critical
- Proper timestamps help track data modifications

### 5. App Architecture
- Separation of concerns (models, services, screens, widgets)
- Singleton pattern for database access
- Reusable components improve code maintainability

---

## Code Quality and Best Practices

### Implemented Practices
- ✅ Singleton pattern for database management
- ✅ Model class with toMap() and fromMap() conversions
- ✅ Proper error handling with try-catch blocks
- ✅ Input validation before database operations
- ✅ Meaningful comments and documentation
- ✅ Consistent naming conventions
- ✅ Separation of concerns (MVC pattern)
- ✅ Responsive UI with loading states
- ✅ User-friendly error messages

### Code Structure
- **Models:** Post class with proper serialization
- **Services:** DatabaseHelper with all CRUD operations
- **Screens:** Separated into Home, Add, Edit screens
- **Widgets:** Reusable PostCard component
- **Main:** Clean app setup with Material Design

---

## Dependencies Explanation

| Dependency | Version | Purpose | Justification |
|-----------|---------|---------|---------------|
| sqflite | ^2.3.0 | SQLite database operations | Essential for local data persistence |
| path_provider | ^2.1.1 | File system directory access | Platform-specific database path handling |
| intl | ^0.19.0 | Date/time formatting | Localization and consistent date display |
| Flutter Material | built-in | UI components | Standard Flutter Material Design |

---

## Submission Checklist

- [x] Source code uploaded to GitHub repository
- [x] All CRUD operations implemented and tested
- [x] Exception handling implemented
- [x] README created with lab report format
- [x] Code is well-documented with comments
- [x] App tested on emulator/device
- [x] Screenshots placeholder ready for insertion
- [x] Project structure is clean and organized

---

## GitHub Repository

**Repository Link:** https://github.com/lewis1011/Lab5-New-.git

**Commit History:**
- Initial project setup with Flutter structure
- Created database helper and Post model
- Implemented CRUD operations
- Created UI screens (Home, Add, Edit)
- Added validation and error handling
- Final testing and optimization

---

## Conclusion

The **Offline Posts Manager** application successfully demonstrates the implementation of local database storage using SQLite in Flutter. The app provides a complete CRUD interface for managing posts offline, with proper error handling, user validation, and an intuitive user interface. All lab requirements have been met, and the application is ready for deployment.

---

## Appendix

### A. Database Schema Visualization
```
posts table:
┌────┬─────────┬──────────────┬────────────────┬────────────────┐
│ id │ title   │ content      │ created_at     │ updated_at     │
├────┼─────────┼──────────────┼────────────────┼────────────────┤
│ 1  │ Title 1 │ Content...   │ 2026-04-25T... │ 2026-04-25T... │
│ 2  │ Title 2 │ Content...   │ 2026-04-25T... │ 2026-04-25T... │
└────┴─────────┴──────────────┴────────────────┴────────────────┘
```

### B. Exception Flow Diagram
```
Database Operation
       ↓
    try {
       ↓
   Execute Query
       ↓
      OK → Return Result
       ├→ UI Update (setState)
       └→ User Feedback (SnackBar)
    }
    catch (e) {
       ↓
   Error Handler
       ↓
   Generate Error Message
       ↓
   Display to User (AlertDialog/SnackBar)
    }
```

### C. Data Flow Architecture
```
UI Layer (Screens)
    ↓
State Management (setState)
    ↓
DatabaseHelper Service
    ↓
sqflite Plugin
    ↓
SQLite Database File
    ↓
Operating System File System
```

---

**Report Prepared by:** Lewis Ngendahimana  
**Date of Submission:** April 25, 2026  
**Lab Session:** Gako Campus - Mobile Computing Lab

---

*This lab report includes all required information about dependencies, exception handling, SQLite implementation, CRUD operations, and Flutter async operations as specified in the lab requirements.*
