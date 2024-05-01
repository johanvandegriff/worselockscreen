# worselockscreen
#After moving my SSD from my framework 13 laptop to my new framework 16, the screen locking inexplicably stopped working. I was planning to reinstall after a few weeks anyway, once ubuntu studio 24.04 was released. So I looked for a quick fix, first trying i3lock and swaylock, but no luck with those on ubuntu. Then I found a few alternatives on github, and tried to install [betterlockscreen](https://github.com/betterlockscreen/betterlockscreen). But I ran into more weird issues and decided to write my own that was as quick and dirty as possible, hence the name "worselockscreen".

#Since I was feeling lazy, I asked chatgpt to write me some code to help get me started. What it gave me didn't even work at all at first, but after some research and iteration it turned into something usable. It even works with the fingerprint reader! It basically operates as an application that tries to prevent anything getting past it, unbinding the exit keys and whatnot. Then I tested every edge case I could think of, disabling Alt+Tab and similar just to cover most of the main ways to switch away from the app. This is a terrible approach to security, but don't worry, I have already done a fresh install and worselockscreen is already in the software graveyard where it belongs.

#Lastly I decided to take things even further and put everything in 1 file. And I mean EVERYTHING. Scroll down. That's the code. In this README. Run it with `./README.md`. That's right, this is a triple-language polyglot: markdown, sh, and python. Why did I do this? I'm glad you asked.

""":"
````sh -c ''

cd `dirname $0`
python -m venv .venv
. .venv/bin/activate
pip install python-pam six
exec python "$0" $@
````
```python
":"""

import tkinter as tk
from tkinter import messagebox, Entry
import pam
import getpass
import os

# Functions to disable/enable anything that might let you get around the lock screen
def disable_super_key():
    # Disable Super key
    os.system("gsettings set org.gnome.mutter overlay-key ''")
    os.system("gsettings set org.gnome.mutter dynamic-workspaces false")
    os.system("gsettings set org.gnome.desktop.wm.preferences num-workspaces 1")
    os.system('''gsettings set org.gnome.desktop.wm.keybindings switch-applications "[]"''')
    os.system('''gsettings set org.gnome.desktop.wm.keybindings switch-applications-backward "[]"''')
    os.system('''gsettings set org.gnome.desktop.wm.keybindings switch-windows "[]"''')
    os.system('''gsettings set org.gnome.desktop.wm.keybindings switch-windows-backward "[]"''')

def enable_super_key():
    # Re-enable Super key to its default value (usually 'Super_L')
    os.system("gsettings set org.gnome.mutter overlay-key 'Super_L'")
    os.system("gsettings set org.gnome.mutter dynamic-workspaces true")
    os.system('''gsettings set org.gnome.desktop.wm.keybindings switch-applications "['<Super>Tab']"''')
    os.system('''gsettings set org.gnome.desktop.wm.keybindings switch-applications-backward "['<Shift><Super>Tab']"''')
    os.system('''gsettings set org.gnome.desktop.wm.keybindings switch-windows "['<Alt>Tab']"''')
    os.system('''gsettings set org.gnome.desktop.wm.keybindings switch-windows-backward "['<Shift><Alt>Tab']"''')

def check_password(event=None):
    password = entry.get()
    p = pam.pam() # Works with password or fingerprint
    user = getpass.getuser() # Get the currently logged-in user
    if p.authenticate(user, password):
        enable_super_key()
        root.quit()
    else:
        messagebox.showerror("Error", "Incorrect password")
        entry.delete(0, tk.END)

# Optional function to bind to Esc, useful for testing
def on_escape(event=None):
    if messagebox.askyesno("Exit", "Do you really want to quit?"):
        enable_super_key()
        root.quit()

# Re-focus the app every 10ms
def keep_focus():
    root.after(10, keep_focus)
    if not root.focus_displayof():
        root.focus_force()

# Set up the root window
root = tk.Tk()
root.attributes('-fullscreen', True) # Make the window full-screen
root.config(bg="black")

# Create a password entry
entry = Entry(root, font=("Helvetica", 24), show='*', justify='center')
entry.pack(expand=True)
entry.focus_set()

# Bind keys
root.bind('<Return>', check_password) # Use the Return (or Enter) key to submit the password
#root.bind('<Escape>', on_escape) # Optionally bind the Escape key to prompt before quitting

# Ensure the window stays on top and cannot be minimized
root.attributes('-topmost', True)
# Prevent the window from being closed or minimized
root.protocol("WM_DELETE_WINDOW", lambda: None)  # Disable the close button

# Disable Super key when application starts
disable_super_key()

# Keep trying to regain focus
#keep_focus()

# Keep the application running
root.mainloop()

"""
```
"""
