# google-signin-android
While working on google or facebook APIs, there are two kinds of resources/data that we can use in our application. We can use public resources without user authentication, for example data about youtube public videos through Youtube Data APIs, but we need user authentication in order to get private data for example user email and sometime we need user authentication in order to perform an action. For that purposes, we can use google sign in API that works on OAuth 2.0 
mechanism. Here's an app in which we have implemented google signin feature using firebase.

### 1. Add dependency into app.gradle
```
implementation 'com.google.firebase:firebase-auth:19.3.2'
implementation 'com.google.android.gms:play-services-auth:18.1.0'
```
you can find latest version on [https://firebase.google.com/docs/auth/android/google-signin](this page)


### 2. Add this xml code in activity’s layout
```
<com.google.android.gms.common.SignInButton
    android:id="@+id/sign_in_button"
    android:layout_width="match_parent"
    android:layout_height="wrap_content" />
```


### 3. Define your Google SignIn request 
Define your request and create a scope in onCreate() method. Default scope is GoogleSignInOptions.DEFAULT_SIGN_IN which is enough for signin. There’s a default web client id in the environment in which you develop android app

```
GoogleSignInOptions gso = new GoogleSignInOptions.Builder(GoogleSignInOptions.DEFAULT_SIGN_IN)
        .requestIdToken(getString(R.string.default_web_client_id))
        .requestEmail()
        .build();
mGoogleSignInClient = GoogleSignIn.getClient(this, gso);
```

### 4. Set dimensions of the signing button and initiate the signIn flow

```
// Set the dimensions of the sign-in button.
final SignInButton signInButton = findViewById(R.id.sign_in_button);
signInButton.setSize(SignInButton.SIZE_STANDARD);

findViewById(R.id.sign_in_button).setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        signIn();
    }
});

```

### 5. Trigger the intent
Create an intent which will open a dialog box having all the emails accounts which will be logged in and we will handle the result in onActivityResult() method. We pass a request code when create intent

```
private void signIn() {
    Intent signInIntent = mGoogleSignInClient.getSignInIntent();
    startActivityForResult(signInIntent, 2);
}

@Override
public void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);

    // Result returned from launching the Intent from GoogleSignInApi.getSignInIntent(...);
    if (requestCode == 2) {
        Task<GoogleSignInAccount> task = GoogleSignIn.getSignedInAccountFromIntent(data);
        try {
            // Google Sign In was successful, authenticate with Firebase
            GoogleSignInAccount account = task.getResult(ApiException.class);
            Log.d("TAGtoken", "firebaseAuthWithGoogle:" + account.getId());
            firebaseAuthWithGoogle(account.getIdToken());
        } catch (ApiException e) {
            // Google Sign In failed, update UI appropriately
            Log.w("TAGFailed", "Google sign in failed", e);
        }
    }
}

```
### 6. Firebase Authenticaiton
We are done with google sign-in, because we are using firebase that's why now we will authenticate for firebase as well. 

```
 private void firebaseAuthWithGoogle(String idToken) {
        AuthCredential credential = GoogleAuthProvider.getCredential(idToken, null);
        mAuth.signInWithCredential(credential)
                .addOnCompleteListener(this, new OnCompleteListener<AuthResult>() {
                    @Override
                    public void onComplete(@NonNull Task<AuthResult> task) {
                        if (task.isSuccessful()) {
                            // Sign in success, update UI with the signed-in user's information
                            Log.d("TAGsigninsuccess", "signInWithCredential:success");
                            FirebaseUser user = mAuth.getCurrentUser();
                            Toast.makeText(GoogleSignInOAuth.this, "Success", Toast.LENGTH_SHORT).show();
                        } else {
                            Toast.makeText(GoogleSignInOAuth.this, "Fail", Toast.LENGTH_SHORT).show();
                            // If sign in fails, display a message to the user.
                            Log.w("TAGsoigninfail", "signInWithCredential:failure", task.getException());
                        }

                    }
                });
    }
```

We received a token after google sign-in, we will pass that token to FirebaseAuth object as credentials for verification


### 7. We don't need to authenticate our user everytime he open the app, that's why we will check user session in onStrat() method and will continue if user will already signed in

```
@Override
    public void onStart() {
        super.onStart();
        // Check if user is signed in (non-null) and update UI accordingly.
        FirebaseUser currentUser = mAuth.getCurrentUser();
        if (currentUser!=null){
            Toast.makeText(this, "do whatever you want", Toast.LENGTH_SHORT).show();
            }
    }
```

I use this code from [https://firebase.google.com/docs/auth/android/google-signin](Firebase)
