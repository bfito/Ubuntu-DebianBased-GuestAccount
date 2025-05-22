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

These methods will help ensure that the **guest user**'s files are automatically deleted after they finish their session. Let me know if you need further help! 😊
