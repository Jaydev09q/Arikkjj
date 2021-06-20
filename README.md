name: CI

on: workflow_dispatch

jobs:
  build:

    runs-on: windows-latest

    steps:
    - name: Download Ngrok
      run: Invoke-WebRequest https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-windows-amd64.zip -OutFile ngrok.zip
    - name: Extract Ngrok Archive
      run: Expand-Archive ngrok.zip
    - name: Auth
      run: .\ngrok\ngrok.exe authtoken $Env:NGROK_AUTH_TOKEN
      env:
        NGROK_AUTH_TOKEN: ${{ secrets.NGROK_AUTH_TOKEN }}
    - name: Enable TS
      run: Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server'-name "fDenyTSConnections" -Value 0
    - name: Firewall for RDP
      run: Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
    - name: Allow RDP Property from Regdit
      run: Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name "UserAuthentication" -Value 1
    - name: Assign User
      run: Set-LocalUser -Name "runneradmin" -Password (ConvertTo-SecureString -AsPlainText "Techguy20" -Force)
    - name: Create Tunnel
      run: Start-Process Powershell -ArgumentList '-Noexit -Command ".\ngrok\ngrok.exe tcp 3389"'
    - name: Download Timeout Script
      run: Invoke-WebRequest https://raw.githubusercontent.com/BladeJag/timeout/main/timeout.ps1 -OutFile timeout.ps1
    - name: Keep Alive
      run: ./timeout.ps1
 
