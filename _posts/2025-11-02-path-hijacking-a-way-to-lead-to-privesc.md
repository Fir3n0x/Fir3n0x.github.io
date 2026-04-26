---
title: "PATH hijacking: A way to lead to PrivEsc"
date: 2025-11-02 16:31:00 +0200
categories: [Pentest]
tags: [hijack, c++, linux, privesc, offensive-security]
image:
  path: /assets/img/posts/path-hijacking-privesc.jpg  # optionnel, image de couverture
---

> Originally published on [Medium](https://medium.com/@corentin_mh/path-hijacking-a-way-to-lead-to-privesc-b33af710bdca).

## What is the PATH variable ?

As you may already know, the PATH variable contains a directory’s list separated by : (Linux/maxOs) or ; (Windows). You can check its content by typing the following command on a terminal :

```bash
echo $PATH
```
When we execute a command like ls, ps or nano, the shell looks for the executable in the order of the directories listed in PATH variable. That’s how the magic trick appears.

## What is the PATH hijacking ?

It exploits writable directories in the system’s PATH environment variable. Therefore, it allows to execute malicious binary instead of legitimate one. How is it exploited by an attacker ?

1.  An attacker finds a writable directory that appears **before** legitimate system directoris in PATH variable.
2.  The attacker create and put a malicious executable, holding the same name as a legitimate binary (e.g. ls, sudo, nano, …) in this writable directory.
3.  When a script (or a user) executes this command, system will launch this malicious binary first, instead of the legitimate one, located farther in the PATH variable.
4.  The malicious binary runs with the permissions of the current user (or higher, if misconfigured).

This technique can be used to gain root privilege. It is not the command itself that is vulnerable, but the environment in which it is executed. If a writable directory is in the PATH, and it is at the top of the list, then any command without an **absolute** path can be hijacked. This become dangerous if a root script or service uses this misconfigured PATH.

## Attack example

I wrote a small script in c++ to retrieve writable directory located in the PATH variable of the current environment.

```c++
bool isOwnedByRoot(const string& dir) {
    struct stat info;
    if (stat(dir.c_str(), &info) != 0) {
        return false;
    }
    return info.st_uid == 0;
}

vector<string> getParsePaths() {
    vector<string> paths;
    const char* pathEnv = getenv("PATH");
    if (!pathEnv) {
        cerr << "Error : PATH not defined." << endl;
        return paths;
    }

    string pathStr(pathEnv);
    istringstream ss(pathStr);
    string token;

    while (getline(ss, token, ':')) {
        paths.push_back(token);
    }
    
    return paths;
}

vector<string> getFilterPaths(const vector<string> paths) {
    vector<string> filterPaths;
    for(auto& path_from_vector : paths) {
        path dir_path = path_from_vector;
        if (exists(dir_path) && is_directory(dir_path)) {
            perms p = status(dir_path).permissions();
            if ((p & perms::owner_write) != perms::none ||
                (p & perms::group_write) != perms::none ||
                (p & perms::others_write) != perms::none) {
                if (!isOwnedByRoot(path_from_vector)) {
                    filterPaths.push_back(path_from_vector);
                }
            }
        }
    }
    return filterPaths;
}

int main() {
    vector<string> pathDirs = getParsePaths();
    vector<string> filterPaths = getFilterPaths(pathDirs);
    cout << "Possible path hijack found (" << filterPaths.size() << "): " << endl;
    for (auto& dir : filterPaths) {
        cout << " - " << dir << endl;
    }
    return 0;
}
```

By running this script on my linux environment, I manage to retrieve writable directories.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*YX8k-Gbt4EAXR7enpipQ3g.png)

Figure 2: Retrieve writable directories from PATH variable

As you can see my script provides me with different directories. If we want to leverage the _ls_ command, we can suppose that the legitimate binary is located in the _/usr/bin_ path. We can locate the ls binary by typing the following command :

![](https://miro.medium.com/v2/resize:fit:621/1*tP_rL4qUy3JGQaFEQfS_Ew.png)

Figure 3: Locate ls binary

As _/home/user/.local/bin_ and _/home/user/.cache/bin_ are located **before** the _/usr/bin_ directory in the PATH variable, we can then leverage both of them to lead to our exploit. Let’s use the first one, **_/home/user/.local/bin_**.

We can create a malicious binary called **_ls_** (as we want to hijack the **ls** command, we need to set our binary with the same name), containing whatever we want. For this example, I am just going to display a message in the terminal. However, this technique can be used to gain root privilege if the command is executed by a user or a service with root permission.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*RslkDqRc79OE2DuUmr8Kng.png)

Figure 4: Create a malicious binary

Now that we have create or malicious _ls_ binary and that we have provided it with execution permission we can execute the ls command in the terminal.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*BvrsLkY5DEgkByUeIz8zAA.png)

Figure 5 : Execute the exploit

As you can see, the attack worked well as the legitimate **ls** binary is not executed anymore by the system environment.

## Mitigations

How to protect from this attack ?

- Never include writable directories in the PATH, especially at the beginning of the string.
- Always use absolute paths in system scripts (e.g. /usr/bin/apt instead of apt).
