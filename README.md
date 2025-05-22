# Ubuntu-DebianBased-GuestAccount
General guest user area creation with temporary session that deletes/clean are data created in session.

To automatically delete all the **guest user's files** after they close their session, you can use a few methods in Linux(DebianBased). One effective approach is to set up a **temporary home directory** for the guest user or configure session-specific file deletion. Here's how to do it:

### **Method 1: Use a Temporary Home Directory with `tmpfs`**

You can set up the **guest user**'s home directory to reside in **tmpfs** (a temporary filesystem) so that any files they create will be automatically deleted when the system reboots or when the session ends.

1. **Create a Temporary Home Directory for the Guest User:**
   First, ensure the guest user does not have a permanent home directory. Create a temporary directory in **`/etc/passwd`** to use **`tmpfs`**.

2. **Edit `/etc/passwd` for the Guest User:**
   Open the **`/etc/passwd`** file to modify the guest user's home directory:

   ```bash
   sudo nano /etc/passwd
   ```

   Find the line corresponding to the **guest** user. It will look something like this:

   ```
   guest:x:1001:1001::/home/guest:/bin/bash
   ```

   Change the home directory to `/dev/shm/guest` (which is in **tmpfs**):

   ```
   guest:x:1001:1001::/dev/shm/guest:/bin/bash
   ```

3. **Create the Temporary Directory in `/dev/shm`:**
   This will create a temporary directory stored in **RAM**, and it will be cleared when the system shuts down:

   ```bash
   sudo mkdir /dev/shm/guest
   sudo chown guest:guest /dev/shm/guest
   ```

4. **Automatically Remove Files After Session Ends:**
   Files in **`/dev/shm`** are lost after a reboot, but you can ensure the **guest user**'s files are deleted at session end by adding cleanup to their **logout** process. To do this, you could configure the **`~/.bash_logout`** file for the guest user:

   * Log in as **guest** and create the **`~/.bash_logout`** file:

     ```bash
     nano /dev/shm/guest/.bash_logout
     ```

   * Add the following line to delete all files in the home directory upon logout:

     ```bash
     rm -rf /dev/shm/guest/*
     ```

### **Method 2: Use `systemd` to Clean Up Files After Logout**

You can also create a **systemd** service to delete the guest user’s files when the session ends.

1. **Create a `systemd` service to clean up files after logout:**

   First, create a service file in the **`/etc/systemd/system/`** directory:

   ```bash
   sudo nano /etc/systemd/system/clean-guest-files.service
   ```

2. **Configure the Service:**
   Add the following content to the service file:

   ```ini
   [Unit]
   Description=Clean up guest user files on logout
   After=multi-user.target

   [Service]
   Type=oneshot
   ExecStart=/bin/rm -rf /dev/shm/guest/*
   User=guest
   StandardOutput=tty
   StandardError=tty

   [Install]
   WantedBy=multi-user.target
   ```

3. **Enable and Start the Service:**
   After saving the service file, enable it so that it starts on boot and runs when the session ends:

   ```bash
   sudo systemctl enable clean-guest-files.service
   sudo systemctl start clean-guest-files.service
   ```

This will automatically clean the **guest user’s files** when their session ends.

### **Method 3: Use `pam_cleanup` Module for Temporary User Sessions**

If you want to create a **temporary guest session** without manually configuring a temporary home directory, you can use the **PAM (Pluggable Authentication Module)** to clean up the user’s files.

1. **Install and Configure `pam_cleanup`:**

   * Install the `pam_cleanup` module for temporary sessions:

     ```bash
     sudo apt install libpam-cleanup
     ```

2. **Configure `pam_cleanup`:**

   * Edit the **PAM configuration** file:

     ```bash
     sudo nano /etc/pam.d/common-session
     ```
   * Add the following line at the end:

     ```bash
     session optional pam_cleanup.so
     ```

This ensures that any files the **guest user** creates are deleted at the end of their session.

### **Conclusion:**

* **Method 1** is a straightforward way to set up a **temporary home directory** in **`tmpfs`** for the guest user, ensuring files are deleted when the system reboots or the session ends.
* **Method 2** gives you more control by using a **`systemd` service** to clean up files after the session ends.
* **Method 3** uses **PAM** to ensure temporary sessions for the guest user, with files cleaned up when the session ends.

Yes, the **temporary files** setup for the **guest user** (using **`tmpfs`** or **`systemd`** cleanup methods) will clean up the **guest's session data**, including **files** created by the guest user. However, **browser data** (such as cache, cookies, browsing history, etc.) typically needs a bit more attention, since browser data is stored in different places.

### **Temporary Files and Session Data**

* **Method 1 (tmpfs)**: By setting the guest user's home directory to **`/dev/shm/guest`** (which is in **RAM**), any files created during the session, including documents, images, and other files, will automatically be **lost** when the system is rebooted or the session ends (because **tmpfs** is a **volatile filesystem**).
* **Method 2 (systemd service)**: If you set up a **`systemd` service** to clean up files on logout, this will ensure that any files created in **`/dev/shm/guest`** are explicitly removed when the guest session ends.

### **Browser Data (Cache, Cookies, History)**

Browsers store their data (such as cache, cookies, and browsing history) in specific **hidden directories** (usually under **`~/.cache`** or **`~/.mozilla`** for Firefox, and **`~/.config/google-chrome`** for Chrome).

To **ensure browser data is cleaned up** after a guest session ends, you can:

1. **Set up Temporary Browser Profiles:**

   * One way to prevent browser data from being saved is by using a **temporary profile** for the **guest user**. Many browsers (like **Chrome** and **Firefox**) allow you to create a new user profile or use **incognito mode** for a session.
   * In **incognito** or **private browsing mode**, the browser doesn’t store any history, cookies, or cache once the session is closed.

2. **Delete Browser Data on Logout:**

   * You can create a cleanup script that specifically **deletes browser data** when the guest user logs out. For instance, add the following to the **`~/.bash_logout`** for the **guest user** (or a system-wide script):

     ```bash
     rm -rf /home/guest/.cache/*
     rm -rf /home/guest/.mozilla/*
     rm -rf /home/guest/.config/google-chrome/*
     ```

3. **Automatic Cleanup with Systemd:**

   * In addition to the **file cleanup** in **`/dev/shm/guest`**, you could add steps to delete specific browser data in the **`systemd` service**. Modify your **`systemd` cleanup script** to include commands to remove browser cache and settings:

     ```bash
     [Service]
     Type=oneshot
     ExecStart=/bin/rm -rf /dev/shm/guest/*
     ExecStart=/bin/rm -rf /home/guest/.cache/*
     ExecStart=/bin/rm -rf /home/guest/.mozilla/*
     ExecStart=/bin/rm -rf /home/guest/.config/google-chrome/*
     ```

### **Important Notes:**

* **Temporary Home Directory**: If you set the **guest user's home directory** to **`/dev/shm/guest`**, this will prevent any persistent data, including browser data, from being saved to disk. However, if the browser stores data outside the guest's home directory (e.g., in **`/tmp`**), additional cleanup will be necessary.
* **Incognito Mode**: Using **incognito** or **private browsing mode** can be an alternative to ensure **browser data** isn’t saved during the session.

### **Conclusion:**

To ensure **all guest session data** (including browser data) is deleted, you can:

* Use a **temporary home directory** (`/dev/shm/guest`) to automatically discard files at reboot or session end.
* Set up **systemd** to clean up specific browser data when the guest session ends.
* Use **incognito** or **private browsing mode** for browser sessions to prevent browser data from being saved.

By combining these methods, you'll ensure that both **file data** and **browser data** are **cleared** when the **guest user** logs out. 
