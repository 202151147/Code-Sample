# Code-Sample

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using TMPro;
using Firebase;
using Firebase.Auth;
using System;
using System.Threading.Tasks;
using Firebase.Extensions;

public class FirebaseController : MonoBehaviour
{

    public GameObject LogIn,SignUp,Profilepage,ForgetPwd,NotifPage;

    public TMP_InputField LogInEmail;
    public TMP_InputField LogInPassword,SignUpEmail,SignUpPassword,SignUpConPassword,SignUpName,ForgetPwdField; 

    public TMP_Text notif_Title_Text,notif_msg_Text,profileName_Text,ProfielEmail_Text;   

    public Toggle RemMe;

    Firebase.Auth.FirebaseAuth auth;
    Firebase.Auth.FirebaseUser user;

    bool isSignIn = false;
    void Start()
    {
      Firebase.FirebaseApp.CheckAndFixDependenciesAsync().ContinueWith(task => {
  var dependencyStatus = task.Result;
  if (dependencyStatus == Firebase.DependencyStatus.Available) {
    // Create and hold a reference to your FirebaseApp,
    // where app is a Firebase.FirebaseApp property of your application class.
       InitializeFirebase();

    // Set a flag here to indicate whether Firebase is ready to use by your app.
  } else {
    UnityEngine.Debug.LogError(System.String.Format(
      "Could not resolve all Firebase dependencies: {0}", dependencyStatus));
    // Firebase Unity SDK is not safe to use here.
  }
});

    }
    public void OpenLogIn()
    {
        LogIn.SetActive(true);
        SignUp.SetActive(false);
        Profilepage.SetActive(false);
        ForgetPwd.SetActive(false);
    }

    public void OpenSignUp()
    {
        LogIn.SetActive(false);
        SignUp.SetActive(true);
        Profilepage.SetActive(false);
        ForgetPwd.SetActive(false);
    }

    public void OpenProfilepage()
    {
        LogIn.SetActive(false);
        SignUp.SetActive(false);
        Profilepage.SetActive(true);
        ForgetPwd.SetActive(false);
    }

    public void OpenForgetpwdpage()
    {
        LogIn.SetActive(false);
        SignUp.SetActive(false);
        Profilepage.SetActive(false);
        ForgetPwd.SetActive(true);
    }
    // Do LogIn
    public void LogInUser()
    {
        if(string.IsNullOrEmpty(LogInEmail.text)&&string.IsNullOrEmpty(LogInPassword.text))
        {
            showMsg("Error","Fields Empty! Please fill all the fields.");
            return;
        }

        SignInExisting(LogInEmail.text,LogInPassword.text);
    }

    // Do SignUp
    public void SignUpUser()
    {
        if(string.IsNullOrEmpty(SignUpEmail.text)&&string.IsNullOrEmpty(SignUpName.text)&&string.IsNullOrEmpty(SignUpPassword.text)&&string.IsNullOrEmpty(SignUpConPassword.text))
        {
            showMsg("Error","Fields Empty! Please fill all the fields.");
            return;
        }

        createUser(SignUpEmail.text,SignUpPassword.text,SignUpName.text,SignUpConPassword.text);
    }

    public void forgotpwd()
    {
        if(string.IsNullOrEmpty(ForgetPwdField.text))
        {
            showMsg("Error","Fields Empty! Please fill all the fields.");
            return;
        }
    }

    private void showMsg(string title,string msg)
    {
        notif_Title_Text.text = "" + title;
        notif_msg_Text.text = "" + msg;
        NotifPage.SetActive(true);
    }

    public void closeNotif()
    {
        notif_Title_Text.text="";
        notif_msg_Text.text="";
        NotifPage.SetActive(false);
    }

    public void LogOut()
    {
        auth.SignOut();
        profileName_Text.text="";
        ProfielEmail_Text.text="";
        OpenLogIn();
    }

    void createUser(string email, string password,string Username,string conPassword)
    {
        auth.CreateUserWithEmailAndPasswordAsync(email, password).ContinueWithOnMainThread(task => {
    if (task.IsCanceled) {
        Debug.LogError("CreateUserWithEmailAndPasswordAsync was canceled.");
        return;
  }
    if (task.IsFaulted) {
        Debug.LogError("CreateUserWithEmailAndPasswordAsync encountered an error: " + task.Exception);

    foreach (Exception exception in task.Exception.Flatten().InnerExceptions)
    {
     
    Firebase.FirebaseException firebaseEx = exception as Firebase.FirebaseException;
    if (firebaseEx != null)
    {
        var errorCode = (AuthError)firebaseEx.ErrorCode;
        showMsg("Error,",GetErrorMessage(errorCode));
    } 
    }
        return;
  }

  // Firebase user has been created.
    Firebase.Auth.FirebaseUser newUser = task.Result;
    Debug.LogFormat("Firebase user created successfully: {0} ({1})",
    newUser.DisplayName, newUser.UserId);

    UpdateUserProfile(Username);
});
    }

    public void SignInExisting(string email, string password)
    {
        auth.SignInWithEmailAndPasswordAsync(email, password).ContinueWithOnMainThread(task => {
  if (task.IsCanceled) {
    Debug.LogError("SignInWithEmailAndPasswordAsync was canceled.");
    return;
  }
  if (task.IsFaulted) {
    Debug.LogError("SignInWithEmailAndPasswordAsync encountered an error: " + task.Exception);
    foreach (Exception exception in task.Exception.Flatten().InnerExceptions)
    {
     
    Firebase.FirebaseException firebaseEx = exception as Firebase.FirebaseException;
    if (firebaseEx != null)
    {
        var errorCode = (AuthError)firebaseEx.ErrorCode;
        showMsg("Error,",GetErrorMessage(errorCode));
    } 
    }
    return;
  }

  Firebase.Auth.FirebaseUser newUser = task.Result;
  Debug.LogFormat("User signed in successfully: {0} ({1})",
      newUser.DisplayName, newUser.UserId);

    profileName_Text.text=""+newUser.DisplayName;
    ProfielEmail_Text.text=""+newUser.Email;
      OpenProfilepage();
});
    }

    void InitializeFirebase() {
  auth = Firebase.Auth.FirebaseAuth.DefaultInstance;
  auth.StateChanged += AuthStateChanged;
  AuthStateChanged(this, null);
}

void AuthStateChanged(object sender, System.EventArgs eventArgs) {
  if (auth.CurrentUser != user) {
    bool signedIn = user != auth.CurrentUser && auth.CurrentUser != null;
    if (!signedIn && user != null) {
      Debug.Log("Signed out " + user.UserId);
    }
    user = auth.CurrentUser;
    if (signedIn) {
      Debug.Log("Signed in " + user.UserId);
      isSignIn = true;
    }
  }
}

void OnDestroy() {
  auth.StateChanged -= AuthStateChanged;
  auth = null;
}

void UpdateUserProfile(string Username)
{
    Firebase.Auth.FirebaseUser user = auth.CurrentUser;
if (user != null) {
  Firebase.Auth.UserProfile profile = new Firebase.Auth.UserProfile {
    DisplayName = Username,
    PhotoUrl = new System.Uri("https://loremflickr.com/320/240"),
  };
  user.UpdateUserProfileAsync(profile).ContinueWith(task => {
    if (task.IsCanceled) {
      Debug.LogError("UpdateUserProfileAsync was canceled.");
      return;
    }
    if (task.IsFaulted) {
      Debug.LogError("UpdateUserProfileAsync encountered an error: " + task.Exception);
      return;
    }

    Debug.Log("User profile updated successfully.");

    showMsg("Alert","Account Successfully Created");
  });
}
}


bool IsSigned=false;

void Update()
{
  if(isSignIn)
  {
    if(!IsSigned)
    {
        IsSigned=true;
        profileName_Text.text=""+user.DisplayName;
    ProfielEmail_Text.text=""+user.Email;
      OpenProfilepage();
    }
  }
}

private static string GetErrorMessage(AuthError errorCode)
{
    var message = "";
    switch (errorCode)
    {
        case AuthError.AccountExistsWithDifferentCredentials:
            message = "Account doesn't exist";
            break;
        case AuthError.MissingPassword:
            message = "Password Not Filled";
            break;
        case AuthError.WeakPassword:
            message = "Password is too Weak";
            break;
        case AuthError.WrongPassword:
            message = "Wrong Password";
            break;
        case AuthError.EmailAlreadyInUse:
            message = "Already Signed In! Try Forgot Password if you forgot your password";
            break;
        case AuthError.InvalidEmail:
            message = "Invalid Email Id entered";
            break;
        case AuthError.MissingEmail:
            message = "Email Id not filled";
            break;
        default:
            message = "Error";
            break;
    }
    return message;
}
}
