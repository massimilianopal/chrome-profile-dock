# Chrome Profile Dock – Automator Source Version (No Icon)

This version lets you recreate the Chrome Profile Dock app using **Automator + AppleScript**, without relying on any executable.

## Requirements

- macOS (tested on Ventura and Sequoia)
- Google Chrome installed
- Chrome user profiles already created

## How to use

1. Open **Automator**.
2. Choose **Application** as type.
3. Add action **Run AppleScript**.
4. Paste the code below.
5. Save the Automator app anywhere (e.g., Desktop or Applications).
6. Double-click to run.

---

## AppleScript code

```applescript
-- Ask user to choose destination folder
set destinationFolder to POSIX path of (choose folder with prompt "Select where you want to create Chrome profile apps:")

-- Define Chrome profiles path
set chromeProfilesPath to POSIX path of (path to home folder) & "Library/Application Support/Google/Chrome/"

-- Find all profile folders
do shell script "mkdir -p " & quoted form of destinationFolder
set profileList to paragraphs of (do shell script "find " & quoted form of chromeProfilesPath & " -type d -name 'Profile*'")

repeat with profilePath in profileList
    set profileName to do shell script "basename " & quoted form of profilePath
    set profileNumber to text ((offset of "Profile " in profileName) + 8) thru (length of profileName) of profileName

    set preferencesFile to profilePath & "/Preferences"
    try
        set email to do shell script "grep -o '\"email\":\"[^\"]*\"' " & quoted form of preferencesFile & " | head -n1 | sed 's/\\\"email\\\":\\\"//;s/\\\"//'"
    on error
        set email to ""
    end try

    if email is not "" then
        set username to do shell script "echo " & quoted form of email & " | cut -d'@' -f1"
        set domain to do shell script "echo " & quoted form of email & " | cut -d'@' -f2 | sed 's/[^a-zA-Z0-9.]/_/g'"

        set appName to username & "_" & domain
        set appPath to destinationFolder & appName & ".app"

        do shell script "mkdir -p " & quoted form of (appPath & "/Contents/MacOS")
        do shell script "mkdir -p " & quoted form of (appPath & "/Contents/Resources")

        -- Create Info.plist
        set infoPlist to "<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n" & ¬
        "<!DOCTYPE plist PUBLIC \"-//Apple//DTD PLIST 1.0//EN\" \"http://www.apple.com/DTDs/PropertyList-1.0.dtd\">\n" & ¬
        "<plist version=\"1.0\">\n" & ¬
        "<dict>\n" & ¬
        "    <key>CFBundleExecutable</key>\n" & ¬
        "    <string>applet</string>\n" & ¬
        "    <key>CFBundleIdentifier</key>\n" & ¬
        "    <string>com.custom.chromeprofile." & appName & "</string>\n" & ¬
        "    <key>CFBundleName</key>\n" & ¬
        "    <string>" & appName & "</string>\n" & ¬
        "    <key>CFBundlePackageType</key>\n" & ¬
        "    <string>APPL</string>\n" & ¬
        "</dict>\n" & ¬
        "</plist>"

        do shell script "echo " & quoted form of infoPlist & " > " & quoted form of (appPath & "/Contents/Info.plist")

        set appletScript to "#!/bin/bash\nopen -na 'Google Chrome' --args --profile-directory='Profile " & profileNumber & "'"
        do shell script "echo " & quoted form of appletScript & " > " & quoted form of (appPath & "/Contents/MacOS/applet")
        do shell script "chmod +x " & quoted form of (appPath & "/Contents/MacOS/applet")

        display notification "Created " & appName
    end if
end repeat

do shell script "open " & quoted form of destinationFolder
display dialog "Profiles created successfully! You can now drag and drop the apps into your Dock for quick access." buttons {"OK"} default button "OK"
