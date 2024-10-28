### **Introduction**

![image](https://github.com/user-attachments/assets/a94306f9-dde3-487e-a398-dad2a80e6c5b)

Single Sign-On (SSO) is a centralized authentication mechanism that allows users to log in once and gain access to multiple, separate applications without needing to authenticate again. Integrating Jenkins, a popular continuous integration/continuous deployment (CI/CD) tool, with WSO2 Identity Server (WSO2 IS), a robust identity and access management solution, via SAML-based SSO provides seamless access for users. By establishing WSO2 IS as the Identity Provider (IdP) for Jenkins, organizations can streamline access management, improve user experience, and enhance security across their DevOps environments.

---

### **Objectives**

The objectives of this setup are:
1. **Centralized Authentication**: Enable WSO2 IS to manage and authenticate users for Jenkins, providing a single login interface.
2. **Streamlined User Experience**: Simplify the user login process by allowing users to authenticate once through WSO2 IS and gain access to Jenkins without further logins.
3. **Enhanced Security**: Protect Jenkins by offloading authentication responsibilities to WSO2 IS, which offers robust security features such as multi-factor authentication, secure session management, and encrypted communications.
4. **Improved User Management**: Utilize WSO2 IS’s advanced user management, including roles, permissions, and groups, to better control user access within Jenkins.
5. **Scalability and Flexibility**: Facilitate easier scalability as the number of users and applications grows, maintaining a consistent and secure authentication process.

---

### **Security Requirements**

Implementing SSO between WSO2 IS and Jenkins introduces several security requirements to ensure safe and secure operations:

1. **SSL/TLS Configuration**: Both Jenkins and WSO2 IS should be configured to use HTTPS (SSL/TLS) to protect data in transit. This ensures that authentication data is encrypted as it flows between Jenkins and WSO2 IS.

2. **Signed and Encrypted Assertions**: Use SAML assertions that are signed and, optionally, encrypted to prevent tampering and interception. Configuring WSO2 IS to sign and encrypt SAML assertions will protect user data transmitted to Jenkins.

3. **Role-Based Access Control (RBAC)**: Set up RBAC within WSO2 IS to enforce strict access control policies, limiting Jenkins access based on user roles and responsibilities.

4. **Session Management**: Enable session management in WSO2 IS to track active user sessions. Configuring single logout (SLO) between Jenkins and WSO2 IS can further ensure that logging out of one application ends the session for all connected applications, enhancing security.

5. **Secure Metadata Exchange**: Ensure that the metadata exchange between WSO2 IS and Jenkins is done securely, verifying that the IdP and Service Provider configurations are trustworthy. This is crucial for validating identity between the two systems.

6. **Multi-Factor Authentication (MFA)**: Implement MFA in WSO2 IS for an added layer of security, especially for users with elevated permissions in Jenkins.

---

The fully detailed guide with step-by-step instructions to set up SSO between WSO2 Identity Server (IS) at `192.168.0.31` and Jenkins at `192.168.0.30` using SAML. 

### Step 1: Set Up WSO2 Identity Server (IS) as the SAML Identity Provider

#### 1.1 Access WSO2 Management Console
1. Open a browser and go to `https://192.168.0.31:9443/carbon`.
2. Log in with the default admin credentials (`admin`/`admin`).

#### 1.2 Create a Service Provider in WSO2 IS
1. Go to **Main > Identity > Service Providers > Add**.
2. Enter a **Service Provider Name** (e.g., `JenkinsSP`).
3. Click **Register** to create the new service provider.

#### 1.3 Configure SAML2 Web SSO for the Service Provider
1. Scroll down to **Inbound Authentication Configuration** and click on **SAML2 Web SSO Configuration**.
2. Click **Configure** and fill out the following details:

    - **Issuer**: Set a unique identifier for Jenkins, such as `jenkins_saml`.
    - **Assertion Consumer URL**: Enter Jenkins’s SAML endpoint, which is typically:
      ```
      https://192.168.0.30/securityRealm/finishLogin
      ```
    - **NameID format**: Choose `urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified`.

3. Under **Select Claim URIs**, select or add the claims (attributes) you want to pass to Jenkins, such as `http://wso2.org/claims/username` and `http://wso2.org/claims/emailaddress`.

4. Enable the following options:
   - **Enable Response Signing**
   - **Enable Assertion Signing**

5. Click **Update** to save these SAML settings.

#### 1.4 Download SAML Metadata from WSO2 IS
1. In the **SAML2 Web SSO Configuration** section, you should see a **Download SAML Metadata** link. 
2. Download and save this XML file, which you’ll need to configure Jenkins.

---

### Step 2: Set Up Jenkins as a SAML Service Provider

#### 2.1 Install the SAML Plugin in Jenkins
1. Go to **Manage Jenkins > Manage Plugins**.
2. In the **Available** tab, search for **SAML Plugin**.
3. Install the **SAML Plugin** and restart Jenkins if prompted.

#### 2.2 Configure SAML Settings in Jenkins
1. Go to **Manage Jenkins > Configure Global Security**.
2. Select **SAML 2.0** as the security realm.
3. Configure the SAML plugin as follows:

   - **IdP Metadata**: Upload the XML metadata file you downloaded from WSO2 IS or paste the metadata URL:
     ```
     https://192.168.0.31:9443/identity/saml2/metadata
     ```

   - **Username Attribute**: Set to the attribute WSO2 IS uses to represent the username, typically `http://wso2.org/claims/username`.
   - **Email Attribute** (Optional): Set to `http://wso2.org/claims/emailaddress`.
   - **Display Name Attribute** (Optional): Set to `http://wso2.org/claims/displayName`.

4. **Advanced Settings** (Optional):
   - **Sign AuthnRequest**: Enable this if you want Jenkins to sign its SAML authentication requests.
   - **Force Authn**: Enable if you want users to authenticate each time they access Jenkins.

5. Click **Save** to apply these SAML settings.

#### 2.3 Manual SAML Configuration in Jenkins `config.xml` (Optional)
If preferred, configure SAML directly in Jenkins's `config.xml` file under Jenkins home directory. Here’s a sample snippet:

```xml
<org.jenkinsci.plugins.saml.SamlSecurityRealm plugin="saml@1.0">
    <idpMetadataFilePath>/path/to/saml-metadata.xml</idpMetadataFilePath>
    <usernameAttributeName>http://wso2.org/claims/username</usernameAttributeName>
    <emailAttributeName>http://wso2.org/claims/emailaddress</emailAttributeName>
    <displayNameAttributeName>http://wso2.org/claims/displayName</displayNameAttributeName>
    <groupsAttributeName>http://wso2.org/claims/groups</groupsAttributeName>
    <maximumAuthenticationLifetime>86400</maximumAuthenticationLifetime>
    <usernameCaseConversion>NONE</usernameCaseConversion>
    <logoutUrl>https://192.168.0.31:9443/oidc/logout</logoutUrl>
    <forceAuthn>false</forceAuthn>
    <authnContextClassRef>urn:oasis:names:tc:SAML:2.0:ac:classes:Password</authnContextClassRef>
</org.jenkinsci.plugins.saml.SamlSecurityRealm>
```

---

### Step 3: Verify the SSO Configuration

1. Restart Jenkins after saving the changes.
2. Go to the Jenkins URL at `https://192.168.0.30`.
3. Click on **Login with SSO**. You should be redirected to the WSO2 IS login page at `https://192.168.0.31`.
4. Enter your WSO2 IS credentials. Upon successful login, you should be redirected back to Jenkins.

---

This setup ensures SSO between Jenkins (at `192.168.0.30`) and WSO2 IS (at `192.168.0.31`), allowing users to log in to Jenkins via WSO2 Identity Server.

### **Conclusion**

Integrating Jenkins with WSO2 Identity Server using SAML-based Single Sign-On (SSO) provides a robust, centralized authentication system that streamlines user access and strengthens security. By enabling SSO, organizations can consolidate user management, enhance the user experience, and enforce advanced security measures across their DevOps ecosystem.

This setup not only secures Jenkins but also scales effectively, supporting seamless user access management as organizational needs evolve. By leveraging WSO2 IS’s rich set of features, such as role-based access control, session management, and multi-factor authentication, organizations can meet compliance and security standards while ensuring users have secure, convenient access to Jenkins and other integrated applications.

--- 

This guide provides a framework to facilitate secure SSO between WSO2 IS and Jenkins, helping organizations improve productivity while adhering to best security practices.
