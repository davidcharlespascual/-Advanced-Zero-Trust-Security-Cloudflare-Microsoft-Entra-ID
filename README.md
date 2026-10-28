# 🛡️ Advanced Zero Trust Security: Cloudflare + Microsoft Entra ID

## 📖 Overview
In this lab, I implemented an additional layer of Zero Trust security on top of my existing Microsoft 365 environment. 

While Entra ID handles the core identity and Conditional Access, I integrated **Cloudflare Zero Trust** to act as a secure perimeter. This ensures that even if a user tries to access a protected internal application, they are forced to authenticate via Entra ID (including MFA) and pass a Cloudflare Access policy before the traffic is allowed through.

## 🏗️ Architecture & Flow
1. **DNS Migration:** Moved authoritative DNS for `ardypascual.site` from Namecheap to Cloudflare.
2. **SSO Integration:** Created an Enterprise Application in Entra ID and connected it to Cloudflare as an Identity Provider (IdP).
3. **Zero Trust Policies:** Created an Access Policy requiring specific user emails to pass authentication.
4. **Application Protection:** Protected a test subdomain (`test.ardypascual.site`) so that only authenticated users can reach it.

---

## 🛠️ Step-by-Step Implementation

### 1. DNS Migration
I successfully migrated the domain to Cloudflare and verified the new nameservers were active.

<img width="1266" height="947" alt="active" src="https://github.com/user-attachments/assets/6fc50e2d-8606-481f-a68a-4ce7f9d2428d" />
<img width="1903" height="947" alt="cloudflare setup" src="https://github.com/user-attachments/assets/2d93cde4-3039-46a3-9956-fda8799d3c7d" />
<img width="1903" height="968" alt="cloudflare setup2" src="https://github.com/user-attachments/assets/4aa3f262-75cd-427d-945c-c0df3c6c5061" />


### 2. Connecting Entra ID as an Identity Provider
I registered a new app in Microsoft Entra ID and configured the API permissions. This acts as the bridge, allowing Cloudflare to trust Microsoft's identity verification.
<img width="1912" height="957" alt="entra app registration1" src="https://github.com/user-attachments/assets/8673bde7-30f5-4541-8137-87a7859f5214" />
<img width="1912" height="957" alt="entraAppregistration2" src="https://github.com/user-attachments/assets/a7a5791f-d5ac-48b8-8b24-94bbe30a2012" />


I then configured the Cloudflare Identity Provider settings with the App ID, Secret, and Directory ID.
<img width="1903" height="968" alt="cloudflare setup3" src="https://github.com/user-attachments/assets/11713db3-3b5c-4684-9fa9-b9ee3cede19b" />
<img width="1903" height="968" alt="cloudflare setup4" src="https://github.com/user-attachments/assets/dae39c62-dc31-4fdb-b47d-d0a6ea7def4e" />
<img width="1903" height="968" alt="cloudflare setup5" src="https://github.com/user-attachments/assets/8d274a94-58fc-4b5f-ab45-10ab80a0cba6" />
<img width="1903" height="968" alt="cloudflare setup6" src="https://github.com/user-attachments/assets/fcd15688-a26d-4177-940c-cb0156040970" />


After configuring the App ID, Secret, and Directory ID, the connection test succeeded, proving Cloudflare could read my Entra ID profile and MFA status.
<img width="1231" height="947" alt="final" src="https://github.com/user-attachments/assets/64ec9ba9-9c5e-48ca-bbe3-091f3ce7684f" />
<img width="1033" height="947" alt="testresult2" src="https://github.com/user-attachments/assets/83cc323a-3e2e-4be2-b639-05a1bf80ecc2" />


### 3. Protecting an Application
I created a self-hosted application in Cloudflare Zero Trust and applied an access policy named `allow admin` that only permits my specific email address. 
<img width="1631" height="972" alt="application config test" src="https://github.com/user-attachments/assets/8a0e0123-5249-460b-9c54-15e8019829cf" />
<img width="1890" height="947" alt="identitySettings" src="https://github.com/user-attachments/assets/baf3794d-de52-4181-9413-91b30443514b" />
<img width="1887" height="958" alt="identitySettings2" src="https://github.com/user-attachments/assets/711d8967-8e8b-4b2e-84f6-744b6d65bdbd" />
<img width="1631" height="972" alt="policy created" src="https://github.com/user-attachments/assets/ebc2ffbb-5469-4d7d-84cc-a8d9751b5d61" />


Finally, I added a DNS "A Record" for the `test` subdomain pointing to a dummy IP (`192.0.2.1`), proxied through Cloudflare, so that Cloudflare Access would intercept all traffic.

<img width="1912" height="957" alt="creating secret" src="https://github.com/user-attachments/assets/7ae8c146-22c9-4ee9-9cb8-847ea97465db" />


---

## 🧪 The Final Test
When navigating to `https://test.ardypascual.site` in an Incognito window, Cloudflare successfully intercepted the request and presented the Zero Trust login portal. 

Only by clicking "Azure AD · Microsoft Entra ID" and passing the MFA prompt was access granted.
<img width="1912" height="957" alt="zerotrust1" src="https://github.com/user-attachments/assets/508a8be6-c05a-4a63-a1e1-e9ee8855f7c4" />
<img width="1864" height="1020" alt="RESULT" src="https://github.com/user-attachments/assets/c1652e24-85fa-40d3-8449-0cdc16c5f59e" />


## 💡 Key Takeaways
*   **Defense in Depth:** Entra ID and Cloudflare Zero Trust work beautifully together. Entra verifies *who* you are, while Cloudflare verifies *if* you should reach the application.
*   **DNS is Critical:** When moving DNS providers, keeping email records (MX, TXT) set to "DNS Only" is mandatory to avoid breaking Microsoft 365.
*   **SSO & MFA:** By enforcing SSO via Entra ID, I can ensure that even if a user's password is compromised, they cannot reach the application without the second factor (MFA).
