## How to check a file is on SSD or HDD

The following implementation does not throw if path is invalid, it will report non-ssd for invalid path/network path

```c++
#include <iostream>
#include <Windows.h>

HANDLE GetVolumeHandleForFile(const wchar_t* filePath)
{
    wchar_t volume_path[MAX_PATH];
    if (!GetVolumePathName(filePath, volume_path, ARRAYSIZE(volume_path)))
        return nullptr;

    wchar_t volume_name[MAX_PATH];
    if (!GetVolumeNameForVolumeMountPoint(volume_path,
        volume_name, ARRAYSIZE(volume_name)))
        return nullptr;

    auto length = wcslen(volume_name);
    if (length && volume_name[length - 1] == L'\\')
        volume_name[length - 1] = L'\0';

    return CreateFile(volume_name, 0,
        FILE_SHARE_READ | FILE_SHARE_WRITE | FILE_SHARE_DELETE,
        nullptr, OPEN_EXISTING, FILE_FLAG_BACKUP_SEMANTICS, nullptr);
}

bool IsFileOnSsd(const wchar_t* file_path)
{
    bool is_ssd{ false };
    HANDLE volume = GetVolumeHandleForFile(file_path);
    if (volume == INVALID_HANDLE_VALUE)
    { return false; /*invalid path! throw?*/ }

    STORAGE_PROPERTY_QUERY query{};
    query.PropertyId = StorageDeviceSeekPenaltyProperty;
    query.QueryType = PropertyStandardQuery;
    DWORD count;
    DEVICE_SEEK_PENALTY_DESCRIPTOR result{};
    if (DeviceIoControl(volume, IOCTL_STORAGE_QUERY_PROPERTY,
        &query, sizeof(query), &result, sizeof(result), &count, nullptr))
    { is_ssd = !result.IncursSeekPenalty; }
    else { /*fails for network path, etc*/ }
    CloseHandle(volume);
    return is_ssd;
}

int main()
{
    std::wcout << IsFileOnSsd(L"C:\\") << "\n";
    std::wcout << IsFileOnSsd(L"D:\\") << "\n";
    return 0;
}
```



## Get UE Game Base Directory

```c++
AbsoluteGameDirectory = FPaths::ConvertRelativePathToFull(FPaths::ProjectDir());
```

## How to import Windows.h in UE

```c++
#include "Windows/PreWindowsApi.h"
#include "Windows/MinWindows.h"
#include "Windows/PostWindowsApi.h"
```

## How to wrap import header files with Window Macros

```c++
#include "Windows/AllowWindowsPlatformTypes.h"
	#include <...>
#include "Windows/HideWindowsPlatformTypes.h"
```



source: [c++ - How to determine whether a file or folder is on SSD or a hard drive? - Stack Overflow](https://stackoverflow.com/questions/50946992/how-to-determine-whether-a-file-or-folder-is-on-ssd-or-a-hard-drive)