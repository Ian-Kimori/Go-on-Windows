# Go-on-Windows

I changed only **Step 5**. It now starts with plain `winget list --id GoLang.Go`, shows the error you'll get, explains it, and then runs the `--source winget` version. Everything else is unchanged.

## Fix: "`go` is not recognized"

Run everything in **PowerShell as Administrator**.

### Step 1: Confirm the error
```powershell
go version
```
If Go isn't working, you'll see this:
```
go : The term 'go' is not recognized as the name of a cmdlet, function, script file, or operable program.
```
That means one of two things: Go isn't installed, or this terminal is still using the PATH it had before the install.

### Step 2: Check whether Go is installed
```powershell
Test-Path "C:\Program Files\Go\bin\go.exe"
```
- `True` → go to **Step 3**
- `False` → go to **Step 5**

### Step 3: Refresh the PATH in this window
```powershell
$env:Path = [Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [Environment]::GetEnvironmentVariable("Path","User")
go version
```
If this prints a version, you're done. If not, go to **Step 4**.

### Step 4: Add Go to your PATH yourself
```powershell
[Environment]::SetEnvironmentVariable("Path", [Environment]::GetEnvironmentVariable("Path","Machine") + ";C:\Program Files\Go\bin", "Machine")
```
Open a new terminal and run `go version`.

### Step 5: Find Go, or install it

**1. Check with winget:**
```powershell
winget list --id GoLang.Go
```
On your network this fails with:
```
Failed when searching source: msstore
An unexpected error occurred while executing the command:
0x8a15005e : The server certificate did not match any of the expected values.
```
**Why it fails:** winget searches every source it knows about, and that includes the **Microsoft Store (`msstore`)**. To reach the Store, winget checks that the server's HTTPS certificate is one it expects. Something on your network (a proxy, VPN or antivirus that inspects HTTPS) swaps in its own certificate, so winget rejects the connection and stops with this error. Go isn't a Store app anyway. It comes from the regular **`winget`** source, which still works fine.

**2. Run it again, limited to the `winget` source:**
```powershell
winget list --id GoLang.Go --source winget
```

**3. Search the drive for `go.exe`:**
```powershell
Get-ChildItem -Path C:\ -Filter go.exe -Recurse -ErrorAction SilentlyContinue | Select-Object -First 5 FullName
```

- **Nothing found:** install Go, then go to **Step 6**.
  ```powershell
  winget install --id GoLang.Go -e --source winget
  ```
  > If winget still fails, download `windows-amd64.msi` from https://go.dev/dl/, run it, keep the defaults, then go to **Step 6**.
- **Found in another folder:** add that folder to your PATH as in Step 4, using the folder that contains `go.exe` in place of `C:\Program Files\Go\bin`.

### Step 6: After installing
Wait for **Successfully installed**, then run these in the same window:

**1. Load the new PATH into this window:**
```powershell
$env:Path = [Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [Environment]::GetEnvironmentVariable("Path","User")
```

**2. Check that Go works:**
```powershell
go version
go env GOROOT GOPATH
```
You should see output like this:
```
go version go1.2x.x windows/amd64
C:\Program Files\Go
C:\Users\<you>\go
```
If `go version` still says not recognized, go back to **Step 4**.

**3. Add the folder for tools you install with `go install`:**
```powershell
$userBin = "$env:USERPROFILE\go\bin"
$userPath = [Environment]::GetEnvironmentVariable("Path","User")
if ($userPath -notlike "*$userBin*") {
    [Environment]::SetEnvironmentVariable("Path", "$userPath;$userBin", "User")
}
```

**4. Run a test program:**
```powershell
cd $env:USERPROFILE
mkdir hello -Force | Out-Null; cd hello
go mod init hello
Set-Content main.go 'package main

import "fmt"

func main() { fmt.Println("Go is working") }'
go run .
```
It should print **Go is working**.

**5. Finish up:**
- Close every PowerShell and terminal window. New windows will pick up Go automatically.
- If you use VS Code, quit it fully and reopen it. Then install the **Go** extension and click **Install All** when it asks to install `gopls` and the other tools.

### Still not working?
Run this and send me the output:
```powershell
Test-Path "C:\Program Files\Go\bin\go.exe"; winget list --id GoLang.Go --source winget
```

**Note:** If you're using the VS Code terminal, close VS Code completely and reopen it. A new terminal tab still uses the old PATH.
