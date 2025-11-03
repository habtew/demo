# The Prophet’s Legacy: Backend API Documentation

Welcome to the official API documentation for **The Prophet’s Legacy**. This document provides a complete, production-ready reference for all backend endpoints, data models, and business logic.

## 1. Overview & Architecture

This API is designed as a stateless backend service to power "The Prophet's Legacy," a gamified Islamic learning application for children. It handles all business logic, user management, content delivery, and progress tracking.

* **Architecture:** A standard RESTful API.
* **Authentication:** Uses JSON Web Tokens (JWT) for all secured endpoints.
* **Data Models:** Employs a two-tier category system for both lessons and challenges to create a structured, level-based learning path.
* **Access Control:** Endpoints are protected based on user roles: `Parent`, `Child`, `Admin`, or `Internal` (for background workers).

---

## 2. Authentication & User Roles

Handles registration, email confirmation, and login for parents and admins.

### 1. Register a Parent Account
* **Endpoint:** `POST /auth/parent/register`
* **Description:** Creates a new parent account. Triggers a confirmation email.
* **Access:** `Public`
* **Request Body:**
    ```json
    {
      "email": "parent@example.com",
      "password": "StrongPass123",
      "name": "Fatima Ibrahim",
      "phone": "+251900000000"
    }
    ```
* **Response Body (201):**
    ```json
    {
      "success": true,
      "message": "Registration successful. Please confirm your email.",
      "parentId": "uuid-p1"
    }
    ```
* **Status Codes:**
    * `201 Created`: Account created successfully.
    * `400 Bad Request`: Invalid email format or weak password.
    * `409 Conflict`: An account with this email already exists.

### 2. Confirm Parent's Email
* **Endpoint:** `POST /auth/parent/confirm`
* **Description:** Verifies a parent's email address using a token sent to them.
* **Access:** `Public`
* **Request Body:**
    ```json
    {
      "token": "confirmationTokenFromEmail"
    }
    ```
* **Response Body (200):**
    ```json
    {
      "success": true,
      "message": "Email verified successfully. we have attached parent id for child linking please use this id to link to parents or search with parent name",
      "parentId": "uuid"
    }
    ```
* **Status Codes:**
    * `200 OK`: Email verified.
    * `400 Bad Request`: Invalid or expired token.

### 3. Login (Parent or Admin)
* **Endpoint:** `POST /auth/login`
* **Description:** Authenticates a user (parent or admin) and returns a JWT.
* **Access:** `Public`
* **Request Body:**
    ```json
    {
      "email": "parent@example.com",
      "password": "Pass123"
    }
    ```
* **Response Body (200):**
    ```json
    {
      "token": "jwt_token_here",
      "user": {
        "id": "uuid-p1",
        "role": "parent",
        "name": "Fatima"
      }
    }
    ```
* **Status Codes:**
    * `200 OK`: Login successful.
    * `401 Unauthorized`: Invalid email or password.

### 4. Request Password Reset
* **Endpoint:** `POST /auth/password-reset/request`
* **Description:** Initiates the password reset process by sending a link to the user's email.
* **Access:** `Public`
* **Request Body:**
    ```json
    {
      "email": "parent@example.com"
    }
    ```
* **Response Body (200):**
    ```json
    {
      "success": true,
      "message": "Password reset link sent."
    }
    ```
* **Status Codes:**
    * `200 OK`: Request received (even if email doesn't exist, to prevent enumeration).
    * `400 Bad Request`: Invalid email format.

### 5. Confirm New Password
* **Endpoint:** `POST /auth/password-reset/confirm`
* **Description:** Sets a new password using the token from the reset email.
* **Access:** `Public`
* **Request Body:**
    ```json
    {
      "token": "resetTokenFromEmail",
      "newPassword": "NewSecurePass456"
    }
    ```
* **Response Body (200):**
    ```json
    {
      "success": true,
      "message": "Password has been reset successfully."
    }
    ```
* **Status Codes:**
    * `200 OK`: Password reset.
    * `400 Bad Request`: Invalid/expired token or weak new password.

---

## 3. Parent, Child, & Preferences

Manages parent/child accounts and user-specific notification settings.

### 6. Get Authenticated Parent's Profile
* **Endpoint:** `GET /parents/me`
* **Description:** Retrieves the profile of the logged-in parent and a summary of their children.
* **Access:** `Parent`
* **Request Body:** None
* **Response Body (200):**
    ```json
    {
      "id": "uuid-p1",
      "name": "Fatima",
      "children": [
        {
          "id": "uuid-c1",
          "name": "Amina",
          "level": 3,
          "stars": 50
        }
      ]
    }
    ```
* **Status Codes:**
    * `200 OK`: Profile retrieved.
    * `401 Unauthorized`: Not logged in as a parent.

### 7. Get Parent Dashboard
* **Endpoint:** `GET /parents/me/dashboard`
* **Description:** Gets summary data for the parent's dashboard view, including their child's leaderboard stats.
* **Access:** `Parent`
* **Request Body:** None
* **Response Body (200):**
    ```json
    {
      "childrenSummary": [
        {
          "id": "c1",
          "name": "Amina",
          "level": 3,
          "stars": 85,
          "leaderboardStats": {
            "dailyRank": 15,
            "weeklyRank": 8,
            "monthlyRank": 3
          }
        }
      ],
      "notifications": [
        {
          "id": "n1",
          "message": "Amina completed a challenge.",
          "read": false
        }
      ]
    }
    ```
* **Status Codes:**
    * `200 OK`: Dashboard data retrieved.
    * `401 Unauthorized`: Not logged in as a parent.

### 8. Register a Child Profile
* **Endpoint:** `POST /children/register`
* **Description:** Creates a new child profile under the parent's account.
* **Access:** `Parent`
* **Request Body:**
    ```json
    {
      "parentId": "uuid-p1",
      "username": "AminaStar",
      "displayName": "Amina",
      "ageGroup": "8-10",
      "avatar": "url_or_id_of_avatar"
    }
    ```
* **Response Body (201):**
    ```json
    {
      "childId": "uuid-c1",
      "message": "Child profile created."
    }
    ```
* **Status Codes:**
    * `201 Created`: Child profile created.
    * `400 Bad Request`: Missing fields or invalid age group.
    * `409 Conflict`: Username already taken.

### 9. Get Authenticated Child's Full Profile
* **Endpoint:** `GET /children/me`
* **Description:** Retrieves the detailed profile of the logged-in child.
* **Access:** `Child`
* **Request Body:** None
* **Response Body (200):**
    ```json
    {
      "id": "uuid-c1",
      "displayName": "Amina",
      "level": 3,
      "stars": 85,
      "streaks": 5,
      "avatarUrl": "url_to_avatar_image"
    }
    ```
* **Status Codes:**
    * `200 OK`: Profile retrieved.
    * `401 Unauthorized`: Not logged in as a child.

### 10. Get User Notification Preferences
* **Endpoint:** `GET /users/me/notification-prefs`
* **Description:** Retrieves notification preferences (tips, reminders, quiet hours) for the authenticated user (parent or child).
* **Access:** `Parent`, `Child`
* **Request Body:** None
* **Response Body (200):**
    ```json
    {
      "pushTips": true,
      "pushReminders": true,
      "locale": "ar",
      "quietHours": {
        "start": "21:00",
        "end": "07:00",
        "timezone": "Africa/Addis_Ababa"
      }
    }
    ```
* **Status Codes:**
    * `200 OK`: Preferences retrieved.
    * `401 Unauthorized`: Not logged in.

### 11. Update User Notification Preferences
* **Endpoint:** `PUT /users/me/notification-prefs`
* **Description:** Updates the notification settings for the authenticated user.
* **Access:** `Parent`, `Child`
* **Request Body:**
    ```json
    {
      "pushTips": false,
      "quietHours": {
        "start": "22:00"
      }
    }
    ```
* **Response Body (200):**
    ```json
    {
      "success": true,
      "message": "Preferences updated."
    }
    ```
* **Status Codes:**
    * `200 OK`: Preferences updated.
    * `400 Bad Request`: Invalid time format.
    * `401 Unauthorized`: Not logged in.

### 12. Mark Parent Notifications as Read
* **Endpoint:** `POST /parents/me/notifications/read`
* **Description:** Marks a list of notifications as read to clear them from the parent's view.
* **Access:** `Parent`
* **Request Body:**
    ```json
    {
      "notificationIds": [
        "n1",
        "n2"
      ]
    }
    ```
* **Response Body (200):**
    ```json
    {
      "success": true
    }
    ```
* **Status Codes:**
    * `200 OK`: Notifications marked as read.
    * `401 Unauthorized`: Not logged in as a parent.

---

## 4. Lessons & Categories

Handles the delivery of educational content (lessons).

### 13. Get All Lesson Categories
* **Endpoint:** `GET /lesson-categories`
* **Description:** Retrieves the main categories for lessons (e.g., "Prophet’s Description").
* **Access:** `Child`
* **Parameters:** `level` (optional, integer) - Filters categories by a specific level.
* **Request Body:** None
* **Response Body (200):**
    ```json
    {
      "categories": [
        {
          "id": "cat-l1",
          "name": "Prophet’s Description",
          "level": 1
        },
        {
          "id": "cat-l2",
          "name": "Virtues of Good Deeds",
          "level": 1
        }
      ]
    }
    ```
* **Status Codes:**
    * `200 OK`: Categories retrieved.
    * `401 Unauthorized`: Not logged in as a child.

### 14. Get Lessons within a Category
* **Endpoint:** `GET /lessons`
* **Description:** Retrieves a list of lessons, filtered by a specific category and level.
* **Access:** `Child`
* **Parameters:**
    * `categoryId` (required, string): The UUID of the lesson category.
    * `level` (required, integer): The level number.
* **Request Body:** None
* **Response Body (200):**
    ```json
    {
      "lessons": [
        {
          "id": "uuid-l1",
          "title": "The Prophet’s Face",
          "imageUrl": "url_to_image"
        }
      ]
    }
    ```
* **Status Codes:**
    * `200 OK`: Lessons retrieved.
    * `400 Bad Request`: Missing required parameters.
    * `401 Unauthorized`: Not logged in as a child.

### 15. Get a Single Lesson's Detail
* **Endpoint:** `GET /lessons/:id`
* **Description:** Retrieves the full detail (text, audio, video) for one specific lesson.
* **Access:** `Child`
* **Parameters:** `id` (required, string): The UUID of the lesson.
* **Request Body:** None
* **Response Body (200):**
    ```json
    {
      "id": "l1",
      "title": "The Prophet’s Face",
      "description": "The Messenger of Allah ﷺ had the most beautiful face...",
      "audioUrl": "url_to_audio_file",
      "imageUrl": "url_to_image_file"
    }
    ```
* **Status Codes:**
    * `200 OK`: Lesson retrieved.
    * `401 Unauthorized`: Not logged in as a child.
    * `404 Not Found`: Lesson with this ID does not exist.

### 16. Mark a Lesson as Complete
* **Endpoint:** `POST /lessons/:id/complete`
* **Description:** Marks a lesson as complete, awards stars, and returns the child's new progress summary. Triggers internal checks for achievements and parent notifications.
* **Access:** `Child`
* **Parameters:** `id` (required, string): The UUID of the lesson.
* **Request Body:**
    ```json
    {
      "childId": "uuid-c1"
    }
    ```
* **Response Body (200):**
    ```json
    {
      "success": true,
      "starsEarned": 5,
      "celebration": {
        "sfxId": "s1"
      },
      "newProgress": {
        "currentLevel": 1,
        "levelProgress": {
          "completedInLevel": 8,
          "totalInLevel": 10,
          "percentage": 80
        },
        "totalStars": 155,
        "currentStreak": 6
      }
    }
    ```
* **Status Codes:**
    * `200 OK`: Lesson marked complete.
    * `401 Unauthorized`: Not logged in as a child.
    * `404 Not Found`: Lesson not found.
    * `409 Conflict`: Lesson already completed by this child.

---

## 5. Challenges & Attempt Sessions

Handles the "Duolingo-style" gamified quiz sessions.

### 17. Get All Challenge Categories for a Level
* **Endpoint:** `GET /challenge-categories`
* **Description:** Retrieves challenge categories, showing if they are locked based on level progression.
* **Access:** `Child`
* **Parameters:**
    * `level` (required, integer): The level number to fetch.
    * `childId` (required, string): The child's UUID to check unlock status.
* **Request Body:** None
* **Response Body (200):**
    ```json
    {
      "categories": [
        {
          "id": "cat-c1",
          "name": "Word Ordering",
          "level": 1,
          "isLocked": false,
          "questionCount": 3,
          "unlockRequirement": "Complete all challenges in the previous level to unlock."
        },
        {
          "id": "cat-c2",
          "name": "Picture Matching",
          "level": 2,
          "isLocked": true,
          "questionCount": 5,
          "unlockRequirement": "Complete all challenges in Level 1 to unlock."
        }
      ]
    }
    ```
* **Status Codes:**
    * `200 OK`: Categories retrieved.
    * `400 Bad Request`: Missing parameters.
    * `401 Unauthorized`: Not logged in as a child.

### 18. Start a Challenge Session
* **Endpoint:** `POST /attempt-sessions`
* **Description:** Creates a session for a child to attempt a series of questions.
* **Access:** `Child`
* **Request Body:**
    ```json
    {
      "childId": "uuid-c1",
      "challengeCategoryId": "cat-c1"
    }
    ```
* **Response Body (201):**
    ```json
    {
      "sessionId": "uuid-s1",
      "message": "Session started.",
      "questionCount": 3
    }
    ```
* **Status Codes:**
    * `201 Created`: Session started.
    * `401 Unauthorized`: Not logged in.
    * `403 Forbidden`: Challenge category is locked for this child.

### 19. Get the Next Question in a Session
* **Endpoint:** `GET /attempt-sessions/:sessionId/next-question`
* **Description:** Fetches the next question in the sequence for an ongoing attempt session.
* **Access:** `Child`
* **Parameters:** `sessionId` (required, string): The UUID of the attempt session.
* **Request Body:** None
* **Response Body (200):**
    ```json
    {
      "questionId": "uuid-q1",
      "type": "multiple_choice",
      "question": "What was the first battle in Islam?",
      "options": [
        "Badr",
        "Uhud",
        "Hunain"
      ],
      "timeLimitSec": 30
    }
    ```
* **Status Codes:**
    * `200 OK`: Question retrieved.
    * `401 Unauthorized`: Not logged in.
    * `404 Not Found`: Session ended or all questions answered.

### 20. Submit an Answer
* **Endpoint:** `POST /attempt-sessions/:sessionId/answers`
* **Description:** Submits an answer and stores its correctness and time taken.
* **Access:** `Child`
* **Parameters:** `sessionId` (required, string): The UUID of the attempt session.
* **Request Body:**
    ```json
    {
      "questionId": "uuid-q1",
      "answer": "Badr",
      "timeTakenSec": 12
    }
    ```
* **Response Body (200):**
    ```json
    {
      "correct": true,
      "score": 10
    }
    ```
* **Status Codes:**
    * `200 OK`: Answer processed.
    * `401 Unauthorized`: Not logged in.
    * `404 Not Found`: Session or question not found.
    * `409 Conflict`: Answer for this question already submitted.

### 21. Finish a Challenge Session
* **Endpoint:** `POST /attempt-sessions/:sessionId/finish`
* **Description:** Concludes the session, calculates the final score (including penalties), awards stars, and returns the new progress state. Triggers internal checks for achievements, level progression, and parent notifications.
* **Access:** `Child`
* **Parameters:** `sessionId` (required, string): The UUID of the attempt session.
* **Request Body:** None
* **Response Body (200):**
    ```json
    {
      "sessionId": "uuid-s1",
      "finalScore": 75,
      "scoring": {
        "baseScore": 90,
        "timePenalty": -5,
        "repeatAttemptPenalty": -10
      },
      "starsEarned": 4,
      "passed": true,
      "celebration": {
        "sfxId": "s2",
        "animationId": "a1"
      },
      "newProgress": {
        "currentLevel": 1,
        "levelProgress": {
          "completedInLevel": 9,
          "totalInLevel": 10,
          "percentage": 90
        },
        "totalStars": 159,
        "currentStreak": 6
      }
    }
    ```
* **Status Codes:**
    * `200 OK`: Session finished and scored.
    * `401 Unauthorized`: Not logged in.
    * `404 Not Found`: Session not found.

---

## 6. Progress, Gamification, & Reminders

Handles progress tracking, leaderboards, and personal reminders.

### 22. Get Child's Progress Summary
* **Endpoint:** `GET /progress/me/summary`
* **Description:** Retrieves the complete progress summary for the authenticated child's dashboard.
* **Access:** `Child`
* **Request Body:** None
* **Response Body (200):**
    ```json
    {
      "currentLevel": 1,
      "levelProgress": {
        "completedInLevel": 7,
        "totalInLevel": 10,
        "percentage": 70
      },
      "totalStars": 150,
      "currentStreak": 5,
      "achievementsUnlocked": 4,
      "totalAchievements": 20
    }
    ```
* **Status Codes:**
    * `200 OK`: Summary retrieved.
    * `401 Unauthorized`: Not logged in as a child.

### 23. Get Leaderboard
* **Endpoint:** `GET /leaderboard`
* **Description:** Retrieves the pre-calculated leaderboard for a specific time period.
* **Access:** `Child`, `Parent`
* **Parameters:** `period` (required, string): `daily`, `weekly`, or `monthly`.
* **Request Body:** None
* **Response Body (200):**
    ```json
    {
      "period": "weekly",
      "entries": [
        {
          "rank": 1,
          "childName": "Yusuf",
          "stars": 150
        }
      ]
    }
    ```
* **Status Codes:**
    * `200 OK`: Leaderboard retrieved.
    * `401 Unauthorized`: Not logged in.

### 24. Get All Favorite Lessons
* **Endpoint:** `GET /children/me/favorites`
* **Description:** Retrieves all favorite lessons for the child.
* **Access:** `Child`
* **Request Body:** None
* **Response Body (200):**
    ```json
    {
      "favorites": [
        {
          "lessonId": "uuid-l1",
          "title": "The Prophet’s Face"
        }
      ]
    }
    ```
* **Status Codes:**
    * `200 OK`: Favorites retrieved.
    * `401 Unauthorized`: Not logged in as a child.

### 25. Add a Lesson to Favorites
* **Endpoint:** `POST /children/me/favorites`
* **Description:** Adds a lesson to the child's favorites.
* **Access:** `Child`
* **Request Body:**
    ```json
    {
      "lessonId": "uuid-l1"
    }
    ```
* **Response Body (201):**
    ```json
    {
      "success": true,
      "message": "Added to favorites."
    }
    ```
* **Status Codes:**
    * `201 Created`: Favorite added.
    * `401 Unauthorized`: Not logged in.
    * `404 Not Found`: Lesson not found.

### 26. Remove a Lesson from Favorites
* **Endpoint:** `DELETE /children/me/favorites/:lessonId`
* **Description:** Removes a lesson from the child's favorites.
* **Access:** `Child`
* **Parameters:** `lessonId` (required, string): The UUID of the lesson.
* **Request Body:** None
* **Response Body (200):**
    ```json
    {
      "success": true,
      "message": "Removed from favorites."
    }
    ```
* **Status Codes:**
    * `200 OK`: Favorite removed.
    * `401 Unauthorized`: Not logged in.
    * `404 Not Found`: Favorite not found.

### 27. Get All Personal Reminders
* **Endpoint:** `GET /children/me/reminders`
* **Description:** Retrieves all personal reminders set by the logged-in child.
* **Access:** `Child`
* **Request Body:** None
* **Response Body (200):**
    ```json
    {
      "reminders": [
        {
          "id": "uuid-r1",
          "title": "Qiyam al-Layl",
          "time": "03:00"
        }
      ]
    }
    ```
* **Status Codes:**
    * `200 OK`: Reminders retrieved.
    * `401 Unauthorized`: Not logged in.

### 28. Create a Personal Reminder
* **Endpoint:** `POST /children/me/reminders`
* **Description:** Creates a new personal reminder (triggers local notifications).
* **Access:** `Child`
* **Request Body:**
    ```json
    {
      "title": "Morning Adhkar",
      "time": "06:30",
      "timezone": "Africa/Addis_Ababa",
      "repeat": "daily",
      "soundRef": "default_chime.mp3",
      "enabled": true
    }
    ```
* **Response Body (201):**
    ```json
    {
      "reminderId": "uuid-r2",
      "message": "Reminder created."
    }
    ```
* **Status Codes:**
    * `201 Created`: Reminder created.
    * `400 Bad Request`: Invalid time or timezone.
    * `401 Unauthorized`: Not logged in.

### 29. Update a Personal Reminder
* **Endpoint:** `PUT /children/me/reminders/:id`
* **Description:** Updates an existing personal reminder.
* **Access:** `Child`
* **Parameters:** `id` (required, string): The UUID of the reminder.
* **Request Body:**
    ```json
    {
      "enabled": false,
      "time": "03:30"
    }
    ```
* **Response Body (200):**
    ```json
    {
      "success": true,
      "message": "Reminder updated."
    }
    ```
* **Status Codes:**
    * `200 OK`: Reminder updated.
    * `401 Unauthorized`: Not logged in.
    * `404 Not Found`: Reminder not found.

### 30. Delete a Personal Reminder
* **Endpoint:** `DELETE /children/me/reminders/:id`
* **Description:** Deletes a personal reminder.
* **Access:** `Child`
* **Parameters:** `id` (required, string): The UUID of the reminder.
* **Request Body:** None
* **Response Body (200):**
    ```json
    {
      "success": true,
      "message": "Reminder deleted."
    }
    ```
* **Status Codes:**
    * `200 OK`: Reminder deleted.
    * `401 Unauthorized`: Not logged in.
    * `404 Not Found`: Reminder not found.

### 31. Get All Global Reminders (Admin)
* **Endpoint:** `GET /admin/reminders`
* **Description:** (Admin) Retrieves all global, admin-managed reminders.
* **Access:** `Admin, child, parents`
* **Request Body:** None
* **Response Body (200):**
    ```json
    {
      "globalReminders": [
        {
          "id": "uuid-gr1",
          "title": "Fasting (Mon/Thu)"
        }
      ]
    }
    ```
* **Status Codes:**
    * `200 OK`: Reminders retrieved.
    * `401 Unauthorized`: 

### 32. Create a Global Reminder (Admin)
* **Endpoint:** `POST /admin/reminders`
* **Description:** (Admin) Creates a new global reminder template.
* **Access:** `Admin`
* **Request Body:**
    ```json
    {
      "title": "Morning Adhkar",
      "message": "Start your day with remembrance...",
      "repeat": "daily",
      "locale": "en",
      "active": true
    }
    ```
* **Response Body (201):**
    ```json
    {
      "reminderId": "uuid-gr2",
      "message": "Global reminder created."
    }
    ```
* **Status Codes:**
    * `201 Created`: Reminder created.
    * `400 Bad Request`: Missing fields.
    * `401 Unauthorized`: Not an admin.

---

## 7. Social, System, & Internal

Handles media sharing, health checks, and internal system triggers.

### 33. Render a Shareable Asset
* **Endpoint:** `POST /share/render`
* **Description:** Creates a job to render a shareable asset (image/video). Ensures no PII.
* **Access:** `Child`
* **Request Body:**
    ```json
    {
      "sourceType": "lesson",
      "sourceId": "uuid-l1",
      "format": "image"
    }
    ```
* **Response Body (202):**
    ```json
    {
      "shareId": "uuid-sh1",
      "status": "pending"
    }
    ```
* **Status Codes:**
    * `202 Accepted`: Job created and queued.
    * `400 Bad Request`: Invalid source type.
    * `401 Unauthorized`: Not logged in.

### 34. Get Status of Shareable Asset
* **Endpoint:** `GET /share/:shareId`
* **Description:** Checks the rendering job status and gets the download URL.
* **Access:** `Child`
* **Parameters:** `shareId` (required, string): The UUID of the share job.
* **Request Body:** None
* **Response Body (200):**
    ```json
    {
      "status": "completed",
      "downloadUrl": "url_to_s3_asset"
    }
    ```
* **Status Codes:**
    * `200 OK`: Status retrieved (could be `pending`, `processing`, `completed`, or `failed`).
    * `401 Unauthorized`: Not logged in.
    * `404 Not Found`: Share job not found.

### 35. Track Social Share (Telemetry)
* **Endpoint:** `POST /share/:shareId/track`
* **Description:** Telemetry endpoint to track which platform content was shared to.
* **Access:** `Public`
* **Parameters:** `shareId` (required, string): The UUID of the share job.
* **Request Body:**
    ```json
    {
      "platform": "whatsapp"
    }
    ```
* **Response Body (200):**
    ```json
    {
      "success": true
    }
    ```
* **Status Codes:**
    * `200 OK`: Tracked successfully.
    * `400 Bad Request`: Invalid platform.

### 36. Get System Health
* **Endpoint:** `GET /health`
* **Description:** Checks the health and status of the API.
* **Access:** `Public`
* **Request Body:** None
* **Response Body (200):**
    ```json
    {
      "status": "ok",
      "uptime": "24h"
    }
    ```

### 37. Get API Version
* **Endpoint:** `GET /version`
* **Description:** Returns the current deployed version of the API.
* **Access:** `Public`
* **Request Body:** None
* **Response Body (200):**
    ```json
    {
      "version": "v1.0.0",
      "releaseDate": "2025-10-31"
    }
    ```

### 38. Get Media Upload URL
* **Endpoint:** `POST /media/upload-url`
* **Description:** Gets a presigned URL to upload media (audio, images) to S3/equivalent.
* **Access:** `Admin` (or `Child` for custom reminder sounds)
* **Request Body:**
    ```json
    {
      "filename": "lesson1.mp3",
      "contentType": "audio/mpeg"
    }
    ```
* **Response Body (200):**
    ```json
    {
      "uploadUrl": "presigned_s3_url",
      "assetKey": "unique_asset_key"
    }
    ```
* **Status Codes:**
    * `200 OK`: URL granted.
    * `400 Bad Request`: Invalid file type.
    * `401 Unauthorized`: Not logged in.

---

## 8. Admin: Lesson Management

Full CRUD for lesson content.

### 39. Get All Lesson Categories (Admin)
* **Endpoint:** `GET /admin/lesson-categories`
* **Access:** `Admin`
* **Request Body:** None
* **Response Body (200):**
    ```json
    [
      {
        "id": "cat-l1",
        "name": "Prophet’s Description",
        "level": 1
      }
    ]
    ```

### 40. Create a Lesson Category
* **Endpoint:** `POST /admin/lesson-categories`
* **Access:** `Admin`
* **Request Body:**
    ```json
    {
      "name": "Stories of the Sahaba",
      "level": 2
    }
    ```
* **Response Body (201):**
    ```json
    {
      "categoryId": "cat-l3",
      "message": "Category created."
    }
    ```

### 41. Update a Lesson Category
* **Endpoint:** `PUT /admin/lesson-categories/:id`
* **Access:** `Admin`
* **Parameters:** `id` (required, string): The category UUID.
* **Request Body:**
    ```json
    {
      "name": "Stories of the Companions"
    }
    ```
* **Response Body (200):**
    ```json
    {
      "success": true,
      "message": "Category updated."
    }
    ```

### 42. Delete a Lesson Category
* **Endpoint:** `DELETE /admin/lesson-categories/:id`
* **Access:** `Admin`
* **Parameters:** `id` (required, string): The category UUID.
* **Request Body:** None
* **Response Body (200):**
    ```json
    {
      "success": true,
      "message": "Category deleted."
    }
    ```

### 43. Get All Lessons (Admin)**
* **Endpoint:** `GET /admin/lessons`
* **Access:** `Admin`
* **Parameters:** `page`, `filterByCategoryId`, `sortByLevel`
* **Request Body:** None
* **Response Body (200):**
    ```json
    {
      "lessons": [
        {
          "id": "uuid-l1",
          "title": "The Prophet’s Face",
          "level": 1
        }
      ],
      "totalPages": 10
    }
    ```

### 44. Create a Lesson
* **Endpoint:** `POST /admin/lessons`
* **Access:** `Admin`
* **Request Body:**
    ```json
    {
      "categoryId": "cat-l1",
      "level": 1,
      "title": "The Prophet’s Hair",
      "description": "...",
      "audioUrl": "...",
      "imageUrl": "..."
    }
    ```
* **Response Body (201):**
    ```json
    {
      "lessonId": "uuid-l5",
      "message": "Lesson created."
    }
    ```

### 45. Update a Lesson
* **Endpoint:** `PUT /admin/lessons/:id`
* **Access:** `Admin`
* **Parameters:** `id` (required, string): The lesson UUID.
* **Request Body:**
    ```json
    {
      "title": "The Noble Hair of the Prophet ﷺ",
      "level": 2
    }
    ```
* **Response Body (200):**
    ```json
    {
      "success": true,
      "message": "Lesson updated."
    }
    ```

### 46. Delete a Lesson
* **Endpoint:** `DELETE /admin/lessons/:id`
* **Access:** `Admin`
* **Parameters:** `id` (required, string): The lesson UUID.
* **Request Body:** None
* **Response Body (200):**
    ```json
    {
      "success": true,
      "message": "Lesson deleted."
    }
    ```

---

## 9. Admin: Challenge Management

Full CRUD for challenge categories and their questions.

### 47. Get All Challenge Categories (Admin)
* **Endpoint:** `GET /admin/challenge-categories`
* **Access:** `Admin`
* **Request Body:** None
* **Response Body (200):**
    ```json
    [
      {
        "id": "cat-c1",
        "name": "Word Ordering",
        "level": 1,
        "questionCount": 3,
        "isPublished": true
      }
    ]
    ```

### 48. Create a Challenge Category
* **Endpoint:** `POST /admin/challenge-categories`
* **Access:** `Admin`
* **Request Body:**
    ```json
    {
      "name": "Hadith Arrangement",
      "level": 1,
      "passPercentage": 70
    }
    ```
* **Response Body (201):**
    ```json
    {
      "categoryId": "cat-c3",
      "message": "Challenge Category created. It is currently in draft."
    }
    ```

### 49. Update a Challenge Category
* **Endpoint:** `PUT /admin/challenge-categories/:id`
* **Access:** `Admin`
* **Parameters:** `id` (required, string): The category UUID.
* **Request Body:**
    ```json
    {
      "passPercentage": 80,
      "published": true
    }
    ```
* **Response Body (200 - Success):**
    ```json
    {
      "success": true,
      "message": "Category updated. Status: Published"
    }
    ```
* **Response Body (400 - Failure):**
    ```json
    {
      "success": false,
      "message": "Cannot publish: Category must have at least 3 questions."
    }
    ```

### 50. Delete a Challenge Category
* **Endpoint:** `DELETE /admin/challenge-categories/:id`
* **Access:** `Admin`
* **Parameters:** `id` (required, string): The category UUID.
* **Request Body:** None
* **Response Body (200):**
    ```json
    {
      "success": true,
      "message": "Category and questions deleted."
    }
    ```

### 51. Get All Questions for a Challenge Category
* **Endpoint:** `GET /admin/challenge-categories/:id/questions`
* **Access:** `Admin`
* **Parameters:** `id` (required, string): The category UUID.
* **Request Body:** None
* **Response Body (200):**
    ```json
    [
      {
        "id": "q1",
        "type": "multiple_choice",
        "question": "..."
      }
    ]
    ```

### 52. Add a Question to a Challenge Category
* **Endpoint:** `POST /admin/challenge-categories/:id/questions`
* **Access:** `Admin`
* **Parameters:** `id` (required, string): The category UUID.
* **Request Body:**
    ```json
    {
      "type": "word_ordering",
      "question": "Arrange the words...",
      "options": [
        "Religion",
        "is",
        "advice"
      ],
      "answer": [
        "Religion",
        "is",
        "advice"
      ]
    }
    ```
* **Response Body (201):**
    ```json
    {
      "questionId": "uuid-q5",
      "message": "Question added."
    }
    ```

### 53. Update a Question
* **Endpoint:** `PUT /admin/challenge-categories/:id/questions/:qid`
* **Access:** `Admin`
* **Parameters:**
    * `id` (required, string): category UUID
    * `qid` (required, string): question UUID
* **Request Body:**
    ```json
    {
      "timeLimitSec": 60,
      "question": "Updated text..."
    }
    ```
* **Response Body (200):**
    ```json
    {
      "success": true,
      "message": "Question updated."
    }
    ```

### 54. Delete a Question
* **Endpoint:** `DELETE /admin/challenge-categories/:id/questions/:qid`
* **Access:** `Admin`
* **Parameters:**
    * `id` (required, string): category UUID
    * `qid` (required, string): question UUID
* **Request Body:** None
* **Response Body (200):**
    ```json
    {
      "success": true,
      "message": "Question deleted."
    }
    ```

### 55. Reorder Questions in a Category
* **Endpoint:** `PATCH /admin/challenge-categories/:id/questions/reorder`
* **Access:** `Admin`
* **Parameters:** `id` (required, string): The category UUID.
* **Request Body:**
    ```json
    {
      "orderedQuestionIds": [
        "q3",
        "q1",
        "q2"
      ]
    }
    ```
* **Response Body (200):**
    ```json
    {
      "success": true,
      "message": "Questions reordered."
    }
    ```

### 56. Get All Challenge Question Types
* **Endpoint:** `GET /admin/challenge-question-types`
* **Access:** `Admin`
* **Description:** Fetches the master list of all available question types to populate a dropdown.
* **Request Body:** None
* **Response Body (200):**
    ```json
    {
      "questionTypes": [
        {
          "id": "letter_arrangement",
          "name": "Letter Arrangement"
        },
        {
          "id": "multiple_choice",
          "name": "Multiple Choice"
        },
        {
          "id": "word_ordering",
          "name": "Word Ordering"
        }
      ]
    }
    ```

---

## 10. Admin: Notifications & Assets

Full CRUD for notification templates, campaigns, and media assets.

### 57. Get All Notification Templates
* **Endpoint:** `GET /admin/notification-templates`
* **Access:** `Admin`
* **Request Body:** None
* **Response Body (200):**
    ```json
    [
      {
        "id": "nt1",
        "name": "Morning Adhkar",
        "type": "reminder"
      }
    ]
    ```

### 58. Create a Notification Template
* **Endpoint:** `POST /admin/notification-templates`
* **Access:** `Admin`
* **Request Body:**
    ```json
    {
      "name": "Morning Adhkar Reminder",
      "type": "reminder",
      "content": {
        "en": "Don't forget your morning adhkār! ☀️",
        "ar": "لا تنس أذكار الصباح! ☀️"
      }
    }
    ```
* **Response Body (201):**
    ```json
    {
      "templateId": "uuid-nt1",
      "message": "Template created."
    }
    ```

### 59. Update a Notification Template
* **Endpoint:** `PUT /admin/notification-templates/:id`
* **Access:** `Admin`
* **Parameters:** `id` (required, string): The template UUID.
* **Request Body:**
    ```json
    {
      "content": {
        "en": "New text..."
      }
    }
    ```
* **Response Body (200):**
    ```json
    {
      "success": true,
      "message": "Template updated."
    }
    ```

### 60. Delete a Notification Template
* **Endpoint:** `DELETE /admin/notification-templates/:id`
* **Access:** `Admin`
* **Parameters:** `id` (required, string): The template UUID.
* **Request Body:** None
* **Response Body (200):**
    ```json
    {
      "success": true,
      "message": "Template deleted."
    }
    ```

### 61. Create a Notification Campaign
* **Endpoint:** `POST /admin/notifications/campaigns`
* **Access:** `Admin`
* **Request Body:**
    ```json
    {
      "templateId": "uuid-nt1",
      "target": {
        "ageGroup": [
          "8-10"
        ]
      },
      "schedule": "2025-12-15T09:00:00Z"
    }
    ```
* **Response Body (201):**
    ```json
    {
      "campaignId": "uuid-camp1",
      "message": "Campaign scheduled."
    }
    ```

### 62. Send a Test Notification
* **Endpoint:** `POST /admin/notifications/campaigns/:id/test`
* **Access:** `Admin`
* **Parameters:** `id` (required, string): The campaign UUID.
* **Request Body:**
    ```json
    {
      "targetUserId": "uuid-p1"
    }
    ```
* **Response Body (200):**
    ```json
    {
      "success": true,
      "message": "Test notification sent."
    }
    ```

### 63. Get All Sound Effects (SFX)
* **Endpoint:** `GET /admin/sfx`
* **Access:** `Admin`
* **Request Body:** None
* **Response Body (200):**
    ```json
    [
      {
        "id": "sfx1",
        "name": "Correct Answer",
        "url": "..."
      }
    ]
    ```

### 64. Add a New Sound Effect (SFX)
* **Endpoint:** `POST /admin/sfx`
* **Access:** `Admin`
* **Request Body:**
    ```json
    {
      "name": "Level Up Sound",
      "sfxUrl": "url_to_sfx_file"
    }
    ```
* **Response Body (201):**
    ```json
    {
      "sfxId": "uuid-sfx2",
      "message": "SFX added."
    }
    ```

### 65. Delete a Sound Effect (SFX)
* **Endpoint:** `DELETE /admin/sfx/:id`
* **Access:** `Admin`
* **Parameters:** `id` (required, string): The SFX UUID.
* **Request Body:** None
* **Response Body (200):**
    ```json
    {
      "success": true,
      "message": "SFX deleted."
    }
    ```

### 66. Get All Animations
* **Endpoint:** `GET /admin/animations`
* **Access:** `Admin`
* **Request Body:** None
* **Response Body (200):**
    ```json
    [
      {
        "id": "a1",
        "name": "Star Burst",
        "url": "..."
      }
    ]
    ```

### 67. Add a New Animation
* **Endpoint:** `POST /admin/animations`
* **Access:** `Admin`
* **Request Body:**
    ```json
    {
      "name": "Star Burst",
      "animationUrl": "url_to_animation_file"
    }
    ```
* **Response Body (201):**
    ```json
    {
      "animationId": "uuid-anim2",
      "message": "Animation added."
    }
    ```

### 68. Delete an Animation
* **Endpoint:** `DELETE /admin/animations/:id`
* **Access:** `Admin`
* **Parameters:** `id` (required, string): The animation UUID.
* **Request Body:** None
* **Response Body (200):**
    ```json
    {
      "success": true,
      "message": "Animation deleted."
    }
    ```

### 69. Get All Users (Admin)
* **Endpoint:** `GET /admin/users`
* **Access:** `Admin`
* **Request Body:** None
* **Response Body (200):**
    ```json
    [
      {
        "id": "p1",
        "email": "parent@example.com",
        "childrenCount": 1
      }
    ]
    ```

### 70. Get All Levels (Admin)
* **Endpoint:** `GET /admin/levels`
* **Access:** `Admin`
* **Request Body:** None
* **Response Body (200):**
    ```json
    [
      {
        "level": 1,
        "lessonCount": 10
      }
    ]
    ```

### 71. Get Child's Progress Summary (Admin)
* **Endpoint:** `GET /admin/progress/summary`
* **Access:** `Admin`
* **Parameters:** `childId` (required, string): The UUID of the child.
* **Request Body:** None
* **Response Body (200):**
    ```json
    {
      "childId": "c1",
      "stars": 50,
      "streaks": 7,
      "completedLessons": 10
    }
    ```

### 72. Get All Achievements (Admin)
* **Endpoint:** `GET /admin/achievements`
* **Access:** `Admin`
* **Request Body:** None
* **Response Body (200):**
    ```json
    {
      "achievements": [
        {
          "id": "a1",
          "name": "Prophet’s Description Master",
          "rule": "Complete all lessons in 'Prophet's Description' category."
        }
      ]
    }
    ```

---

## 11. Internal Workers

Endpoints designed to be called by schedulers (e.g., cron jobs) or async event queues, not by the public.

### 73. Internal: Dispatch Notification Campaign
* **Endpoint:** `POST /internal/workers/campaign-dispatch`
* **Access:** `Internal`
* **Request Body:**
    ```json
    {
      "campaignId": "uuid-camp1"
    }
    ```
* **Response Body (200):**
    ```json
    {
      "success": true,
      "notificationsSent": 150
    }
    ```

### 74. Internal: Send Event-Based Parent Notification
* **Endpoint:** `POST /internal/workers/notify-parent`
* **Access:** `Internal`
* **Request Body:**
    ```json
    {
      "parentId": "uuid-p1",
      "eventType": "lesson_completed",
      "message": "Amina just completed 'The Prophet's Smile'!"
    }
    ```
* **Response Body (200):**
    ```json
    {
      "success": true,
      "deliveryStatus": "queued"
    }
    ```

### 75. Internal: Render Shareable Asset
* **Endpoint:** `POST /internal/workers/share-render`
* **Access:** `Internal`
* **Request Body:**
    ```json
    {
      "shareId": "uuid-sh1"
    }
    ```
* **Response Body (200):**
    ```json
    {
      "success": true,
      "outputUrl": "url_to_rendered_asset"
    }
    ```

### 76. Internal: Update Leaderboards
* **Endpoint:** `POST /internal/workers/update-leaderboards`
* **Access:** `Internal`
* **Request Body:**
    ```json
    {
      "period": "daily"
    }
    ```
* **Response Body (200):**
    ```json
    {
      "success": true,
      "period": "daily",
      "entriesCalculated": 500
    }
    ```

### 77. Internal: Check Achievement Unlocks
* **Endpoint:** `POST /internal/workers/check-achievements`
* **Access:** `Internal`
* **Request Body:**
    ```json
    {
      "childId": "uuid-c1"
    }
    ```
* **Response Body (200):**
    ```json
    {
      "success": true,
      "achievementsUnlocked": [
        "a1",
        "a3"
      ]
    }
    ```

### 78. Internal: Update Streaks
* **Endpoint:** `POST /internal/workers/update-streaks`
* **Access:** `Internal`
* **Request Body:**
    ```json
    {}
    ```
* **Response Body (200):**
    ```json
    {
      "success": true,
      "streaksUpdated": 120,
      "streaksReset": 45
    }
    ```

---

## 12. Error Codes

Common HTTP status codes used across the API.

| Code | Status | Description |
| :--- | :--- | :--- |
| `200 OK` | OK | The request was successful. |
| `201 Created` | Created | The resource was successfully created. |
| `202 Accepted` | Accepted | The request was accepted for processing (used for async jobs). |
| `400 Bad Request` | Bad Request | The request was malformed, missing required fields, or had invalid data. |
| `401 Unauthorized` | Unauthorized | No valid JWT was provided. The user must log in. |
| `403 Forbidden` | Forbidden | The user is authenticated but does not have permission to access this resource (e.g., a Child trying to access an Admin endpoint). |
| `404 Not Found` | Not Found | The requested resource (e.g., a specific lesson or user) does not exist. |
| `409 Conflict` | Conflict | The request could not be completed due to a conflict with the current state of the resource (e.g., email or username already exists). |
| `500 Internal Server Error` | Internal Server Error | An unexpected error occurred on the server. |
