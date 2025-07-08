# Android Quiz App Development Prompt

## Project Overview
Create a comprehensive Android quiz application with Firebase backend for competitive exam practice. The app should support 31 different papers with 100 questions each, user management, real-time scoring, and leaderboards.

## Backend Requirements (Firebase Firestore)

### 1. Database Schema

#### Collection: `questions`
```
Document Structure:
{
  questionNo: number,
  paper: string, // "Paper1", "Paper2", ... "Paper31"
  question: string,
  optionA: string,
  optionB: string,
  optionC: string,
  optionD: string,
  correctOption: string, // "A", "B", "C", or "D"
  selectedOption: string // User's selected option (initially empty)
}
```

#### Collection: `paperMenu`
```
Document Structure:
{
  papername: string // "Paper1", "Paper2", ... "Paper31"
}
```

#### Collection: `users`
```
Document Structure:
{
  uid: string, // Firebase Auth UID
  name: string,
  mobile: string, // Format: "+91XXXXXXXXXX"
  email: string,
  membershipNumber: string, // 6-digit number
  createdAt: timestamp,
  emailVerified: boolean,
  phoneVerified: boolean
}
```

#### Collection: `userResults`
```
Document Structure:
{
  userId: string,
  paperName: string,
  score: number, // Out of 100
  totalQuestions: number, // Always 100
  correctAnswers: number,
  wrongAnswers: number,
  completedAt: timestamp,
  timeTaken: number // In minutes
}
```

#### Collection: `userProgress`
```
Document Structure:
{
  userId: string,
  paperName: string,
  currentQuestionIndex: number,
  selectedAnswers: map, // questionNo -> selectedOption
  startedAt: timestamp,
  lastUpdated: timestamp
}
```

### 2. Firebase Setup Requirements

#### Authentication
- Enable Email/Password authentication
- Enable Phone authentication
- Implement email verification
- Implement phone number verification with +91 country code

#### Firestore Security Rules
```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Users can only access their own data
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
    
    // Questions are read-only for authenticated users
    match /questions/{questionId} {
      allow read: if request.auth != null;
    }
    
    // Paper menu is read-only for authenticated users
    match /paperMenu/{paperId} {
      allow read: if request.auth != null;
    }
    
    // User results - users can only access their own
    match /userResults/{resultId} {
      allow read, write: if request.auth != null && 
        resource.data.userId == request.auth.uid;
    }
    
    // User progress - users can only access their own
    match /userProgress/{progressId} {
      allow read, write: if request.auth != null && 
        resource.data.userId == request.auth.uid;
    }
  }
}
```

## Frontend Requirements (Android)

### 3. User Authentication & Registration

#### Registration Screen
- **Name Field**: Required, minimum 2 characters
- **Email Field**: 
  - Required
  - Validate email format using Android patterns
  - Check for proper email structure (user@domain.com)
- **Mobile Number Field**:
  - Required
  - Exactly 10 digits
  - Auto-prefix with +91
  - Validate Indian mobile number format
- **Membership Number Field**:
  - Required
  - Exactly 6 digits
  - Numeric only

#### Validation Implementation
```kotlin
// Email validation
fun isValidEmail(email: String): Boolean {
    return android.util.Patterns.EMAIL_ADDRESS.matcher(email).matches()
}

// Mobile validation
fun isValidMobile(mobile: String): Boolean {
    return mobile.length == 10 && mobile.all { it.isDigit() }
}

// Membership number validation
fun isValidMembershipNumber(membershipNumber: String): Boolean {
    return membershipNumber.length == 6 && membershipNumber.all { it.isDigit() }
}
```

#### Authentication Flow
1. User registers with email and phone
2. Send email verification
3. Send OTP to phone number (+91 prefix)
4. Both verifications must be completed before access
5. Store user data in Firestore after successful verification

### 4. Main App Features

#### Dashboard Screen
- Welcome message with user name
- Grid/List of 31 papers (Paper1 to Paper31)
- Show progress indicator for each paper
- Access to leaderboard
- Logout option

#### Paper Selection Screen
- Display available papers from `paperMenu` collection
- Show completion status for each paper
- Show best score for completed papers
- "Start Quiz" or "Resume Quiz" buttons

#### Quiz Screen
- Display current question number (e.g., "Question 5 of 100")
- Show question text
- Four option buttons (A, B, C, D)
- Previous/Next navigation buttons
- Question indicator showing attended questions
- Auto-save progress after each question
- Timer display (optional)

#### Quiz Features Implementation
```kotlin
class QuizViewModel : ViewModel() {
    fun saveAnswer(questionNo: Int, selectedOption: String) {
        // Save to userProgress collection
        // Update selectedAnswers map
    }
    
    fun getNextQuestion(): Question? {
        // Fetch next question from questions collection
        // Filter by current paper
    }
    
    fun calculateScore(): Int {
        // Compare selectedOption with correctOption
        // Return total correct answers
    }
}
```

#### Result Screen
- Display final score (X out of 100)
- Show percentage
- Display correct/wrong answer count
- Show time taken
- "View Leaderboard" button
- "Reset All Selections" button (for all papers)
- "Take Another Paper" button

#### Leaderboard Screen
- Display top 20 performers across all papers
- Columns: Rank, Name, Score, Paper Name
- Sort by highest score first
- Real-time updates using Firestore listeners

#### Reset Functionality
```kotlin
fun resetAllSelections() {
    // Clear selectedOption field for all questions of all papers for current user
    // Delete userProgress documents for current user
    // Show confirmation dialog before reset
}
```

### 5. Technical Implementation Details

#### Dependencies (build.gradle)
```kotlin
dependencies {
    // Firebase
    implementation 'com.google.firebase:firebase-auth:22.1.2'
    implementation 'com.google.firebase:firebase-firestore:24.7.1'
    implementation 'com.google.firebase:firebase-analytics:21.3.0'
    
    // UI Components
    implementation 'androidx.recyclerview:recyclerview:1.3.1'
    implementation 'androidx.cardview:cardview:1.0.0'
    implementation 'com.google.android.material:material:1.9.0'
    
    // Architecture Components
    implementation 'androidx.lifecycle:lifecycle-viewmodel-ktx:2.6.2'
    implementation 'androidx.lifecycle:lifecycle-livedata-ktx:2.6.2'
    implementation 'androidx.navigation:navigation-fragment-ktx:2.6.0'
    implementation 'androidx.navigation:navigation-ui-ktx:2.6.0'
    
    // Image Loading
    implementation 'com.github.bumptech.glide:glide:4.15.1'
}
```

#### Key Activities/Fragments Structure
```
├── MainActivity (Host for navigation)
├── AuthenticationFragment
├── RegistrationFragment
├── EmailVerificationFragment
├── PhoneVerificationFragment
├── DashboardFragment
├── PaperSelectionFragment
├── QuizFragment
├── ResultFragment
├── LeaderboardFragment
└── ProfileFragment
```

#### Data Models
```kotlin
data class Question(
    val questionNo: Int = 0,
    val paper: String = "",
    val question: String = "",
    val optionA: String = "",
    val optionB: String = "",
    val optionC: String = "",
    val optionD: String = "",
    val correctOption: String = "",
    var selectedOption: String = ""
)

data class User(
    val uid: String = "",
    val name: String = "",
    val mobile: String = "",
    val email: String = "",
    val membershipNumber: String = "",
    val createdAt: Timestamp? = null,
    val emailVerified: Boolean = false,
    val phoneVerified: Boolean = false
)

data class UserResult(
    val userId: String = "",
    val paperName: String = "",
    val score: Int = 0,
    val totalQuestions: Int = 100,
    val correctAnswers: Int = 0,
    val wrongAnswers: Int = 0,
    val completedAt: Timestamp? = null,
    val timeTaken: Int = 0
)
```

### 6. UI/UX Guidelines

#### Design Specifications
- Material Design 3 components
- Primary color scheme for branding
- Responsive design for different screen sizes
- Progress indicators for long operations
- Loading states for network operations
- Error handling with user-friendly messages

#### Question Display
- Clear, readable fonts (minimum 16sp)
- Proper spacing between options
- Visual feedback for selected options
- Color coding for navigation (attempted questions)

#### Accessibility
- Content descriptions for screen readers
- Minimum touch target sizes (48dp)
- High contrast colors
- Support for text scaling

### 7. Performance Optimization

#### Data Loading Strategy
- Paginate questions (load 10-20 at a time)
- Cache questions locally using Room database
- Implement offline capability for better UX
- Use Firebase offline persistence

#### Memory Management
- Implement proper lifecycle management
- Use weak references for listeners
- Clean up resources in onDestroy()

### 8. Additional Features

#### Analytics & Monitoring
- Track user engagement with Firebase Analytics
- Monitor app crashes with Firebase Crashlytics
- Track quiz completion rates
- Monitor user retention

#### Notifications
- Send reminders for incomplete quizzes
- Notify about new papers or updates
- Achievement notifications

### 9. Testing Strategy

#### Unit Tests
- Validation logic testing
- Score calculation testing
- Data model testing

#### Integration Tests
- Firebase authentication flow
- Firestore read/write operations
- Navigation testing

#### UI Tests
- Complete quiz flow testing
- Registration process testing
- Leaderboard functionality testing

### 10. Deployment Checklist

#### Pre-launch
- [ ] Set up Firebase project
- [ ] Configure authentication providers
- [ ] Upload initial question data (3100 questions)
- [ ] Set up Firestore security rules
- [ ] Test all user flows
- [ ] Performance testing
- [ ] Security testing

#### Post-launch
- [ ] Monitor app performance
- [ ] Track user feedback
- [ ] Monitor Firebase usage and costs
- [ ] Regular data backups
- [ ] Update questions periodically

## Implementation Priority

### Phase 1 (Core Features)
1. Firebase setup and authentication
2. User registration and verification
3. Basic quiz functionality
4. Score calculation and results

### Phase 2 (Enhanced Features)
1. Leaderboard implementation
2. Progress tracking
3. Reset functionality
4. UI/UX improvements

### Phase 3 (Advanced Features)
1. Offline capability
2. Analytics integration
3. Push notifications
4. Performance optimizations

This comprehensive prompt provides a complete roadmap for developing the Android quiz application with all specified requirements. The implementation should follow Android best practices and ensure a smooth, user-friendly experience.