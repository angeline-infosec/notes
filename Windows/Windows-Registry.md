# Windows Registry

Not to be confused with a "model registry" in AI, which is a versioned store of trained models.

## What it is

The registry is a hierarchical database where Windows and applications store configuration settings, covering the operating system, drivers, services, user preferences and installed software. Your guess is close. It holds most low-level configuration, though not literally every setting, and some entries are used internally and are not meant to be edited. You can browse it with Registry Editor (`regedit`) or the `reg` command.

## Structure

- **Keys** work like folders and can contain subkeys and values.
- **Values** hold the actual data, each with a name, a type (such as a string, a number or binary data) and the data itself.
- **Root keys** are the five top-level branches:

| Root key | Short | What it holds |
|---|---|---|
| HKEY_LOCAL_MACHINE | HKLM | Settings for the whole computer, such as hardware, services and installed software |
| HKEY_CURRENT_USER | HKCU | Settings for the user currently logged in (an alias of that user's branch under HKU) |
| HKEY_USERS | HKU | Settings for all loaded user profiles, plus the default profile |
| HKEY_CLASSES_ROOT | HKCR | File associations and COM registrations (a combined view of machine and user settings) |
| HKEY_CURRENT_CONFIG | HKCC | The hardware profile currently in use |

- **Hives:** Microsoft describes a hive as a logical group of keys, subkeys and values with supporting files, loaded when Windows starts or a user logs in. Each user's settings live in a file called `NTUSER.DAT`, and the system hives (SYSTEM, SOFTWARE, SAM and SECURITY) are stored in `C:\Windows\System32\config`.

## Why a SOC analyst cares

- **Persistence:** programs listed in the `Run` keys start at logon, for example `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` and the HKLM equivalent. This is ATT&CK technique T1547.001 (Registry Run Keys / Startup Folder), the same one mapped in the Windows Logging for SOC lab.
- **Tampering with settings:** attackers can change registry values to alter services or weaken security tools.
- **Credential theft:** saving the SAM and SYSTEM hives is a common way to extract password hashes, which is why commands that export these hives deserve attention.
- **Logging:** Sysmon records registry activity as Event ID 12 (key created or deleted), 13 (value set) and 14 (key or value renamed). The Security log equivalent is 4657 (a registry value was modified), which only appears if auditing is enabled, and it is off by default.
- **Reading a Sysmon Event 13:** check the process that made the change (`Image`), the registry path (`TargetObject`) and the new value (`Details`). A Run key set by a process running from `Temp` or `Downloads` is a red flag.

## Safety

Editing the registry incorrectly can break Windows. Export or back up a key first, and practise only on a test VM.

## Sources

Hive and structure details follow Microsoft Learn's registry documentation (primary). The Sysmon event IDs and the hive file locations are from general knowledge and the Windows Logging for SOC room, and I have not rechecked them against Microsoft's current Sysmon documentation.
