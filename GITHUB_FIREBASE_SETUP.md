# 🚀 GitHub + Firebase Setup Guide

This guide shows you how to deploy your Firebase-based AI Study Notes app to GitHub Pages with proper security.

## 🔐 **Security Options for GitHub**

Since GitHub Pages doesn't support environment variables, you have several secure options:

### **Option 1: Public Firebase Config (Recommended for GitHub)**
- Firebase API keys are meant to be public
- Security comes from Firestore rules, not hiding the config
- This is the standard approach for client-side Firebase apps

### **Option 2: Build-time Configuration**
- Use GitHub Actions to inject config during build
- More complex but keeps config out of repository

### **Option 3: Netlify/Vercel (Recommended)**
- Better hosting with environment variable support
- Free tier available

## 🚀 **Option 1: Public Config (Easiest)**

### Step 1: Create Firebase Project

1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Click "Create a project"
3. Name it "ai-study-notes" (or your choice)
4. Enable Google Analytics (optional)
5. Click "Create project"

### Step 2: Enable Authentication

1. In Firebase Console, go to "Authentication"
2. Click "Get started"
3. Go to "Sign-in method" tab
4. Enable "Email/Password"
5. Click "Save"

### Step 3: Create Firestore Database

1. Go to "Firestore Database"
2. Click "Create database"
3. Choose "Start in test mode"
4. Select a location close to your users
5. Click "Done"

### Step 4: Get Firebase Configuration

1. Go to Project Settings (gear icon)
2. Scroll down to "Your apps"
3. Click "Web" icon (`</>`)
4. Register your app with name "AI Study Notes"
5. Copy the Firebase configuration object

### Step 5: Update Your App

Create a new file `firebase-app-github.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI Study Notes Generator - GitHub Version</title>
    <!-- Your existing CSS here -->
</head>
<body>
    <!-- Your existing HTML here -->

    <!-- Firebase SDK -->
    <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-auth-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-firestore-compat.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/js/all.min.js"></script>

    <script>
        // Firebase Configuration - Replace with your actual config
        const firebaseConfig = {
            apiKey: "YOUR_ACTUAL_API_KEY_HERE",
            authDomain: "your-project.firebaseapp.com",
            projectId: "your-project-id",
            storageBucket: "your-project.appspot.com",
            messagingSenderId: "123456789",
            appId: "1:123456789:web:abcdef123456"
        };

        // App ID for Firestore collections
        const APP_ID = "ai_study_notes";

        // Initialize Firebase
        firebase.initializeApp(firebaseConfig);
        const auth = firebase.auth();
        const db = firebase.firestore();

        // Your existing JavaScript code here...
    </script>
</body>
</html>
```

### Step 6: Secure Firestore Rules

In Firebase Console, go to "Firestore Database" → "Rules" and replace with:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Users can only access their own data
    match /artifacts/{appId}/users/{userId}/user_data/{document} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

### Step 7: Deploy to GitHub

1. **Create Repository**:
   ```bash
   git init
   git add .
   git commit -m "Add Firebase AI Study Notes app"
   git remote add origin https://github.com/yourusername/ai-study-notes.git
   git push -u origin main
   ```

2. **Enable GitHub Pages**:
   - Go to repository Settings → Pages
   - Source: "Deploy from a branch"
   - Branch: `main`
   - Save

3. **Your app will be live at**: `https://yourusername.github.io/ai-study-notes/firebase-app-github.html`

## 🔧 **Option 2: GitHub Actions with Environment Variables**

### Step 1: Create GitHub Secrets

1. Go to your repository Settings → Secrets and variables → Actions
2. Add these repository secrets:
   - `FIREBASE_API_KEY`
   - `FIREBASE_AUTH_DOMAIN`
   - `FIREBASE_PROJECT_ID`
   - `FIREBASE_STORAGE_BUCKET`
   - `FIREBASE_MESSAGING_SENDER_ID`
   - `FIREBASE_APP_ID`

### Step 2: Create GitHub Actions Workflow

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout
      uses: actions/checkout@v4
      
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        
    - name: Inject Firebase Config
      run: |
        # Create the app with injected config
        cat > firebase-app-github.html << 'EOF'
        <!DOCTYPE html>
        <html lang="en">
        <head>
            <meta charset="UTF-8">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
            <title>AI Study Notes Generator</title>
            <!-- Your CSS here -->
        </head>
        <body>
            <!-- Your HTML here -->
            
            <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-app-compat.js"></script>
            <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-auth-compat.js"></script>
            <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-firestore-compat.js"></script>
            <script src="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/js/all.min.js"></script>
            
            <script>
                const firebaseConfig = {
                    apiKey: "${{ secrets.FIREBASE_API_KEY }}",
                    authDomain: "${{ secrets.FIREBASE_AUTH_DOMAIN }}",
                    projectId: "${{ secrets.FIREBASE_PROJECT_ID }}",
                    storageBucket: "${{ secrets.FIREBASE_STORAGE_BUCKET }}",
                    messagingSenderId: "${{ secrets.FIREBASE_MESSAGING_SENDER_ID }}",
                    appId: "${{ secrets.FIREBASE_APP_ID }}"
                };
                
                const APP_ID = "ai_study_notes";
                
                firebase.initializeApp(firebaseConfig);
                const auth = firebase.auth();
                const db = firebase.firestore();
                
                // Your JavaScript code here...
            </script>
        </body>
        </html>
        EOF
        
    - name: Deploy to GitHub Pages
      uses: peaceiris/actions-gh-pages@v3
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        publish_dir: ./
```

## 🌟 **Option 3: Netlify (Recommended)**

### Why Netlify is Better for Firebase:

- ✅ **Environment Variables**: Built-in support
- ✅ **Automatic Deploys**: On every push
- ✅ **Custom Domains**: Free SSL
- ✅ **Better Performance**: CDN included
- ✅ **Form Handling**: Built-in features

### Netlify Setup:

1. **Connect Repository**:
   - Go to [Netlify](https://netlify.com)
   - Click "New site from Git"
   - Connect your GitHub repository

2. **Set Environment Variables**:
   - Go to Site settings → Environment variables
   - Add all your Firebase config variables

3. **Deploy**: Netlify automatically deploys on every push

## 🔒 **Security Best Practices**

### 1. Firebase API Keys are Public by Design

- Firebase API keys are meant to be public
- Security comes from Firestore rules, not hiding the config
- This is the standard approach for client-side Firebase apps

### 2. Firestore Rules are Your Security

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /artifacts/{appId}/users/{userId}/user_data/{document} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

### 3. Additional Security Measures

- Enable Firebase App Check (optional)
- Set up Firebase Security Rules
- Use Firebase Authentication
- Enable HTTPS only

## 📱 **Testing Your Deployment**

1. **Create Account**: Register on your deployed app
2. **Add Data**: Create some notes and flashcards
3. **Test Cross-Device**: Login from another device
4. **Verify Sync**: Check that data appears on all devices

## 🎯 **Recommended Approach**

For GitHub Pages, I recommend **Option 1** (Public Config) because:

- ✅ **Simplest to set up**
- ✅ **Standard Firebase practice**
- ✅ **Secure with proper Firestore rules**
- ✅ **No complex build processes**
- ✅ **Works immediately**

**Your Firebase app will work perfectly on GitHub Pages with true cross-device access!** 🚀✨
