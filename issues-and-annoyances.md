### General usability and PowerShell support

  - Windows PowerShell 5.1 support
  - PowerShell cmdlets not feature-equivalent to winget DOS CLI:
    - missing --no-upgrade
    - missing pin management
    - more ...
  - Isn't officially supported on Windows Server, except Windows Server 2025 but ONLY with the Desktop Experience
  - Can't be run on Server 2019 "Core" at all (https://github.com/microsoft/winget-cli/discussions/2361#discussioncomment-8134429)
  - [Install-WingetPackage always force-reinstalls packages on every run](https://github.com/microsoft/winget-cli/issues/3455)
  - [Can't uninstall winget / DesktopAppInstaller once installed](https://github.com/microsoft/winget-cli/discussions/2361#discussioncomment-8062672)
  - [Parameters unexpectedly behave differently](https://github.com/microsoft/winget-cli/issues/4313)
  
### Private package repositories

  - Still no authentication support (https://github.com/microsoft/winget-cli-restsource/issues/100)
  - [MotW bug](https://github.com/microsoft/winget-cli/issues/4046)
  - Correlation of installed programs still spotty due to unfixed https://github.com/microsoft/winget-cli-restsource/issues/59
  - [ProductCode queried incorrectly](https://github.com/microsoft/winget-cli/issues/2558)
  - Unclear how backwards/forwards compatibility should be handled: https://github.com/microsoft/winget-cli-restsource/issues/194 & https://github.com/microsoft/winget-cli/issues/2023
  
### Automation-friendliness of the winget DOS CLI (as an alternative to the unusable PowerShell cmdlets)

  - [winget CLI doesn't officially run in WinRM](https://github.com/microsoft/winget-cli/issues/256) (only with hacks)
  - [winget CLI doesn't officially run in ssh](https://github.com/microsoft/winget-cli/issues/513) (only with hacks)
  - [Telling winget to install a package that's currently installed is an error](https://github.com/microsoft/winget-cli/issues/4262)
  - [The output of `winget export` cannot be captured](https://github.com/microsoft/winget-cli/issues/4267)
  - [Animations and spinners cannot be turned off](https://github.com/microsoft/winget-cli/issues/3494)

### Other bugs and inconveniences

  - Random new issues come up often e.g. [--scope machine breaking](https://github.com/microsoft/winget-cli/issues/4410) or https://github.com/microsoft/winget-cli/issues/4880
  - https://github.com/microsoft/winget-cli/issues/4679

