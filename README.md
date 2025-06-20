# Using Authgear in Webflow: A Step-by-Step Guide

With Webflow sunsetting its native User Accounts feature, many developers face the challenge of finding a secure, robust, and easy-to-integrate solution. Now, imagine you're building a loyalty program website—a vibrant digital haven where members effortlessly log in to check their personalized content and track their reward points. Your community is excited to explore exclusive benefits and see their progress, but with the traditional authentication method gone, how do you ensure a flawless, secure experience?

Enter Authgear. This powerful tool steps in to bridge the gap left by Webflow. Authgear’s generous free tier and versatile suite of features make it the ideal replacement for managing user authentication on your loyalty program site. Whether you’re a seasoned developer or just beginning your journey, integrating Authgear ensures your members can seamlessly log in, interact with exclusive content, and feel secure every step of the way.

In this guide, you’ll embark on a step-by-step process to integrate Authgear into your Webflow website. By the end, you’ll have learned to:

*   Create an Authgear project and an application.
*   Add Login, Logout, and Sign Up buttons that show/hide correctly.
*   Implement the core authentication logic for your Webflow site.
*   Display standard and custom user information in a protected section.
*   Create and manage gated content visible only to logged-in users.

You can check the end product of this guide in [here](https://authgear-webflow-demo.webflow.io/), you can also clone the Webflow project [here](https://webflow.com/made-in-webflow/website/authgear-webflow-demo).

### Prerequisites

*   An active **Authgear account**. If you don't have one, you can [sign up for free](https://www.authgear.com/cloud).
*   A **Webflow account** and a project you want to add authentication to.

---

## Step 1: Create Your Authgear Project

First, we'll create a project, which acts as a container for your applications and users.

1.  After signing into the [Authgear Portal](https://portal.authgear.com/), you will be prompted to create a new project.
2.  Enter a **Project Name** (e.g., `My Webflow Site`). 
3.  Complete the endpoint domain to form your unique Authgear endpoint: `https://<your-project-name>.authgear.cloud`. Click **"Next"**.

![Project Name](/assets/images/fig1-project-name.png)

4.  Choose how you would like your end-users to login. In this tutorial, make sure we choose Email here.

![Choose how end-users login](/assets/images/fig2-email-password.png)

5.  Add branding by uploading logo and choosing the colors you like.

![Choose a Credential Combination](/assets/images/fig2a-branding.png)


## Step 2: Create and Configure an Application

Now that you have a project, you need to create an application within it.

1.  In the Authgear Portal, navigate to **Applications** in the left-hand menu.
2.  Click the **+ Add Application** button in the top toolbar.
3.  In the dialog, enter an **Application Name** (e.g., "Webflow Site") and select the **Single Page Application** type. Click **"Save"**.

![Select Single Page Application](/assets/images/fig3-spa.png)

4.  On the next screen, Authgear shows tutorials for various frameworks. Click **Next** to skip this and go to your application's configuration page.
5.  You will now be on the application configuration page. Under the **URIs** section, find the **Authorized Redirect URIs** field.
6.  Enter the root URL of your published Webflow site (e.g., `https://my-awesome-site.webflow.io/`). **Note that the trailing "/" in the above URLs must be included.**
7.  Click **Save** at the bottom of the page.
8.  Finally, make a note on your **Client ID** and **Endpoint**. You will need these values in a later step.

![Copy Client ID and Endpoint, enter URL of Webflow site](/assets/images/fig4-auth-redirect-uri.png)

---

## Step 3: Add Custom Styles to Webflow

1.  In your Webflow project, go to **Site Settings > Custom Code**.
2.  In the **Head Code** section, paste the CSS styles below. These styles prevent a "flash" of content before our script has a chance to run. The script itself will be added to the footer in the next step for better page performance.

    ```html
    <!-- Initial Visibility Styles -->
    <style>
      .auth---visible, .auth---invisible {
        visibility: hidden;   
      }
      .hidden {
        display: none !important; 
      }
    </style>
    ```

3.  Click **"Save Changes"**.

---

## Step 4: Add the SDK and Style Authentication Buttons

1.  In the **Webflow Designer**, add three **Button** elements to your page.
2.  Set their text to "Login", "Sign Up", and "Logout".
3.  Assign classes and IDs:
    *   **Login Button**: Class `auth---invisible`, ID `login-btn`
    *   **Sign Up Button**: Class `auth---invisible`, ID `signup-btn`
    *   **Logout Button**: Class `auth---visible`, ID `logout-btn`

4.  Go back to **Site Settings > Custom Code** and paste the following into the **Footer Code** section. This includes the Authgear SDK and the initial logic for your buttons.

    ```javascript
    <!-- Authgear Web SDK -->
    <script src="https://unpkg.com/@authgear/web@2.2.0/dist/authgear-web.iife.js"></script>

    <script>
    const login = () => { console.log("Login button clicked."); };
    const logout = () => { console.log("Logout button clicked."); };
    
    const addClickListeners = () => {
      document.querySelector("#login-btn")?.addEventListener("click", login);
      document.querySelector("#signup-btn")?.addEventListener("click", login);
      document.querySelector("#logout-btn")?.addEventListener("click", logout);
    };
    
    window.onload = () => {
        addClickListeners();
    };
    </script>
    ```

---

## Step 5: Implement the Core Authentication and UI Logic

1.  **Update the Footer Code**: In your Webflow **Footer Code**, replace the contents of the *second* `<script>` tag with the full logic below. **Remember to update the placeholder values with your credentials from Step 2.**

    ```javascript
    // --- Authgear Configuration ---
    let authgearClient = null;
    
    const configureClient = async () => {
        authgearClient = window.authgear.default;
        await authgearClient.configure({
            endpoint: "<YOUR_AUTHGEAR_PROJECT_DOMAIN>",
            clientID: "<YOUR_AUTHGEAR_APP_CLIENT_ID>",
            sessionType: "refresh_token",
        }).then(
            () => { console.log("Authgear client successfully configured!"); },
            (err) => { console.log("Failed to configure Authgear", err); }
        );
    };

    // --- Authentication Functions ---
    const login = () => {
      try {
        console.log("Redirecting to login...");
        const targetURL = `${window.location.origin}/`;
        authgearClient.startAuthentication({ redirectURI: targetURL, prompt: "login" });
      } catch (err) { console.log("Login failed", err); }
    };
    
    const logout = () => {
      try {
        console.log("Logging out");
        authgearClient.logout({
          redirectURI: window.location.origin
        }).then(
          () => { updateUI(); },
          (err) => { console.log("Logout failed", err); }
        );
      } catch (err) { console.log("Log out failed", err); }
    };

    // --- UI Control ---
    const eachElement = (selector, fn) => {
      for (let e of document.querySelectorAll(selector)) { fn(e); }
    };
    
    const updateUI = async () => {
        const isAuthenticated = authgearClient.sessionState === "AUTHENTICATED";
        if (isAuthenticated) {
            eachElement(".auth---invisible", (e) => e.classList.add("hidden"));
            eachElement(".auth---visible", (e) => {
                e.classList.remove("hidden");
                e.style.visibility = "visible";
            });
        } else {
            if (document.body.classList.contains("auth---visible")) { window.location.replace("/"); }
            eachElement(".auth---invisible", (e) => {
                e.classList.remove("hidden");
                e.style.visibility = "visible";
            });
            eachElement(".auth---visible", (e) => e.classList.add("hidden"));
        }
    };

    // --- Event Listeners and Page Load ---
    const addClickListeners = () => {
      document.querySelector("#login-btn")?.addEventListener("click", login);
      document.querySelector("#signup-btn")?.addEventListener("click", login);
      document.querySelector("#logout-btn")?.addEventListener("click", logout);
    };
    
    window.onload = async () => {
      await configureClient();
      addClickListeners();
      try {
        if (window.location.search.includes("code=")) {
            await authgearClient.finishAuthentication();
            window.history.replaceState({}, document.title, window.location.pathname);
        }
      } catch {}
      try {
        await authgearClient.refreshAccessToken();
      } catch {} 
      finally {
        console.log("Authgear Session State:", authgearClient.sessionState);
        updateUI();
      }
    };
    ```

2.  **Create Your First User**:
    1.  **Publish** your Webflow site.
    2.  Open the live site URL and click **"Sign Up"** to create an account.
    3.  After signing up, you will be redirected back, and the "Logout" button should be visible.
    4.  Verify this new user exists in your **Authgear Admin Portal** under **User Management > Users**.

---

## Step 6: Add Elements for Standard User Information

1.  In the **Webflow Designer**, add two **Paragraph** elements to your page.
2.  Assign them IDs and text:
    *   First Paragraph: **ID** `user-email`, Text `Membership Email:`
    *   Second Paragraph: **ID** `user-email-verified`, Text `Verification Status:`
3.  In your **Footer Code**, update the `updateUI` function to fetch and **append** this data.

    ```javascript
    const updateUI = async () => {
        const isAuthenticated = authgearClient.sessionState === "AUTHENTICATED";
        if (isAuthenticated) {
            eachElement(".auth---invisible", (e) => e.classList.add("hidden"));
            eachElement(".auth---visible", (e) => {
                e.classList.remove("hidden");
                e.style.visibility = "visible";
            });
            // --- THIS PART IS NEW ---
            try {
                const userInfo = await authgearClient.fetchUserInfo();
                document.getElementById("user-email").textContent += ` ${userInfo.email}`;
                document.getElementById("user-email-verified").textContent += ` ${userInfo.emailVerified ? "Verified" : "Not Verified"}`;
            } catch (error) { console.error("Failed to fetch user info:", error); }
            // --- END NEW PART ---
        } else { // ... else logic from step 5
            if (document.body.classList.contains("auth---visible")) { window.location.replace("/"); }
            eachElement(".auth---invisible", (e) => {
                e.classList.remove("hidden");
                e.style.visibility = "visible";
            });
            eachElement(".auth---visible", (e) => e.classList.add("hidden"));
        }
    };
    ```

---

## Step 7: Add and Display Custom Attributes

### 7a. Create, Configure, and Assign a Custom Attribute

1.  **Define the Attribute**:
    1.  In the Authgear Admin Portal, navigate to **User Profile > Custom Attributes**.
    2.  Click **"+ Add New Attribute"**.
    3.  Configure it: Name `points_collected`, Type **Number**. Click **"Save"**.
    4.  In the list, set **Token Bearer Access Right** and **End-user Access Right** to **Read-only**.

![Add Custom Attributes](/assets/images/fig5-add-custom-attr.png)
![Set Custom Attributes Access Rights](/assets/images/fig6-set-custom-attr.png)


2.  **Assign a Value to Your User**:
    1.  Navigate to **User Management > Users**, and click on your test user.
    2.  In the **Profile** Tab, scroll down to **Custom Attributes**.
    3.  Enter a number (e.g., `50`) in the **Points Collected** field and click **"Save"**.

![Assign Custom Attributes](/assets/images/fig7-assign-custom-attr.png)

### 7b. Display the Custom Attribute in Webflow

1.  In the **Webflow Designer**, add a new **Paragraph** element to your page.
2.  Give it the **ID** `points-collected` and set its text to `FlowPoints Collected:`.
3.  In your **Footer Code**, update the `updateUI` function again to fetch and display this value.

    ```javascript
    const updateUI = async () => {
        // ...
        if (isAuthenticated) {
            // ...
            try {
                const userInfo = await authgearClient.fetchUserInfo();
                document.getElementById("user-email").textContent += ` ${userInfo.email}`;
                document.getElementById("user-email-verified").textContent += ` ${userInfo.emailVerified ? "Verified" : "Not Verified"}`;
                
                // --- THIS PART IS NEW ---
                const points = userInfo.customAttributes?.points_collected ?? 0;
                document.getElementById("points-collected").textContent += ` ${points}`;
                // --- END NEW PART ---

            } catch (error) { console.error("Failed to fetch user info:", error); }
        }
        // ...
    };
    ```

---

## Step 8: Protect Content

### 8a. Protect a Section of a Page

1.  In the **Webflow Designer**, add a **Div Block** to your page for the "logged out" view.
    *   Give it the class `auth---invisible`.
    *   Inside, add a **Paragraph** with the text "Please login to see this locked content".
    *   Add a **Button** with the text "Login to View" and the class `content-button`.

2.  Add a second **Div Block** for the "logged in" view.
    *   Give it the class `auth---visible`.
    *   **Drag the three paragraph elements** from Steps 6 and 7 inside this div.

3.  Finally, add a click listener for the new `content-button`.

    ```javascript
    const addClickListeners = () => {
      document.querySelector("#login-btn")?.addEventListener("click", login);
      document.querySelector("#signup-btn")?.addEventListener("click", login);
      document.querySelector("#logout-btn")?.addEventListener("click", logout);
      // This line is new
      document.querySelector(".content-button")?.addEventListener("click", login);
    };
    ```

### 8b. Protect an Entire Page

1.  Create a new page in Webflow (e.g., "locked-content").
2.  In the **Navigator** panel, select the **Body** element.
3.  In the **Style Panel (S)**, give the Body the class `auth---visible`. Our script will now automatically redirect any unauthenticated users from this page.

---

## Conclusion and Final Code

Here is the complete, final script to be placed in your Webflow project's **Footer Code**.

```javascript
<!-- Authgear Web SDK -->
<script src="https://unpkg.com/@authgear/web@2.2.0/dist/authgear-web.iife.js"></script>

<script>
let authgearClient = null;

const configureClient = async () => {
    authgearClient = window.authgear.default;
    await authgearClient.configure({
        endpoint: "<YOUR_AUTHGEAR_PROJECT_DOMAIN>",
        clientID: "<YOUR_AUTHGEAR_APP_CLIENT_ID>",
        sessionType: "refresh_token",
    }).then(
        () => { console.log("Authgear client successfully configured!"); },
        (err) => { console.log("Failed to configure Authgear", err); }
    );
};

const login = () => {
  try {
    console.log("Redirecting to login...");
    const targetURL = `${window.location.origin}/`;
    authgearClient.startAuthentication({ redirectURI: targetURL, prompt: "login" });
  } catch (err) { console.log("Login failed", err); }
};

const logout = () => {
  try {
    console.log("Logging out");
    authgearClient.logout({
      redirectURI: window.location.origin
    }).then(
      () => {
        // logged out successfully
        updateUI();
      },
      (err) => {
        // failed to logout
        console.log("Logout failed", err);
      }
    );
  } catch (err) {
    console.log("Log out failed", err);
  }
};

const eachElement = (selector, fn) => {
  for (let e of document.querySelectorAll(selector)) { fn(e); }
};

const updateUI = async () => {
  const isAuthenticated = authgearClient.sessionState === "AUTHENTICATED";

  if (isAuthenticated) {
    eachElement(".auth---invisible", (e) => e.classList.add("hidden"));
    eachElement(".auth---visible", (e) => {
      e.classList.remove("hidden");
      e.style.visibility = "visible";
    });

    try {
      const userInfo = await authgearClient.fetchUserInfo();
      // Clear previous data to prevent duplication on UI refreshes
      document.getElementById("user-email").textContent = 'Membership Email:';
      document.getElementById("user-email-verified").textContent = 'Verification Status:';
      document.getElementById("points-collected").textContent = 'FlowPoints Collected:';
      
      // Append new data
      document.getElementById("user-email").textContent += ` ${userInfo.email}`;
      document.getElementById("user-email-verified").textContent += ` ${userInfo.emailVerified ? "Verified" : "Not Verified"}`;
      const points = userInfo.customAttributes?.points_collected ?? 0;
      document.getElementById("points-collected").textContent += ` ${points}`;
    } catch (error) { console.error("Failed to fetch user info:", error); }

  } else {
    if (document.body.classList.contains("auth---visible")) {
        window.location.replace("/");
    }
    eachElement(".auth---invisible", (e) => {
        e.classList.remove("hidden");
        e.style.visibility = "visible";
    });
    eachElement(".auth---visible", (e) => e.classList.add("hidden"));
  }
};

const addClickListeners = () => {
  document.querySelector("#login-btn")?.addEventListener("click", login);
  document.querySelector("#signup-btn")?.addEventListener("click", login);
  document.querySelector("#logout-btn")?.addEventListener("click", logout);
  document.querySelector(".content-button")?.addEventListener("click", login);
};

window.onload = async () => {
  await configureClient();
  addClickListeners();

  try {
    if (window.location.search.includes("code=")) {
        await authgearClient.finishAuthentication();
        window.history.replaceState({}, document.title, window.location.pathname);
    }
  } catch {}

  // `refreshAccessToken` ensures the session information is up-to-date.
  try {
    await authgearClient.refreshAccessToken();
  } catch {
    // It's normal for this to fail for logged-out users. We can safely ignore the error.
  } finally {
    // Update the UI. This `finally` block runs regardless of whether
    // the user was logged in or not, ensuring the UI is always correct.
    console.log("Authgear Session State:", authgearClient.sessionState);
    updateUI();
  }
};
</script>
```
