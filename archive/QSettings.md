# QSettings

## 1. Persistent storage for application settings.

Location: Stored in platform-specific locations determined by the Qt framework. Typically:
        
- On Windows: Registry or AppData.
- On macOS: ~/Library/Preferences.
- On Linux: ~/.config or ~/.local/share.

Stored Data:
Security settings (e.g., JavaScript enabled, anti-fingerprinting enabled, Tor network enabled, quantum encryption enabled, HTTPS enforced).
The QSettings object is initialized with the organization name "DarkelfBrowser" and application name "Darkelf".

python

self.settings = QSettings("DarkelfBrowser", "Darkelf")

## 2. Environment Variables

Purpose: Storing cryptographic keys securely.

Location: Environment variables are typically session-specific and do not persist across reboots unless explicitly set in the system environment configuration.

Stored Data:
AES key: Stored in the environment variable AES_KEY.
RSA key pair: Stored in the environment variables RSA_PRIVATE_KEY and RSA_PUBLIC_KEY.

python

os.environ['AES_KEY'] = b64encode(aes_key).decode()
os.environ['RSA_PRIVATE_KEY'] = b64encode(private_key).decode()
os.environ['RSA_PUBLIC_KEY'] = b64encode(public_key).decode()

## 3. Cookies

Purpose: Storing session data and user preferences for websites.

Location: Stored by the QWebEngineProfile in a directory determined by the Qt framework.

Stored Data:
All cookies set by websites visited using the browser.

python

profile = QWebEngineProfile.defaultProfile()

## 4. Local Storage (Web Storage)

Purpose: Storing web application data.

Location: Stored by the QWebEngineProfile in a directory determined by the Qt framework.

Stored Data:
Local storage and session storage data used by websites.

python

settings.setAttribute(QWebEngineSettings.LocalStorageEnabled, True)

## 5. HTML5 Storage (IndexedDB, WebSQL, etc.)

Purpose: Storing more complex web application data.
Location: Stored by the QWebEngineProfile in a directory determined by the Qt framework.

## 6. Clearing and Viewing Stored Data

Clear Cookies: Deletes all cookies stored by the browser.

python

profile.cookieStore().deleteAllCookies()

View Cookies: Displays all cookies stored by the browser.

python

profile.cookieStore().loadAllCookies(display_cookies)

The Darkelf browser primarily uses the QSettings mechanism for configuration data, environment variables for cryptographic keys, and the QWebEngineProfile for web data storage such as cookies and local storage. The storage locations are handled by the underlying Qt framework, which ensures they are stored in appropriate, platform-specific locations.
