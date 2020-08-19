# google-signin-android
An app to implement google signin feature into android app using firebase 
### 1. Add dependency into app.gradle

```
implementation 'com.google.firebase:firebase-auth:19.3.2'
implementation 'com.google.android.gms:play-services-auth:18.1.0'
```

### 2. Add this xml code in activity’s layout

```
<com.google.android.gms.common.SignInButton
    android:id="@+id/sign_in_button"
    android:layout_width="match_parent"
    android:layout_height="wrap_content" />

```




### 3. Define your Google SignIn request and create a scope in onCreate() method. Default scope is GoogleSignInOptions.DEFAULT_SIGN_IN which is enough for signin. There’s a default web client id in the environment in which you develop android app

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

### 5. Trigger the intent which will open a dialog box having all the emails accounts which will be logged in and we will handle the result in onActivityResult() method. We pass a request code when create intent

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


### 6. Now we will authenticate 
