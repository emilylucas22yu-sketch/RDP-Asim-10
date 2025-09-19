name: RDP
on: workflow_dispatch

jobs:
  build:
    runs-on: windows-latest
    steps:
    - name: Download Ngrok
      run: |
        Invoke-WebRequest https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-stable-windows-amd64.zip -OutFile ngrok.zip
        Expand-Archive ngrok.zip
        Copy-Item .\ngrok.exe C:\Windows\System32

    - name: Auth Ngrok
      run: ngrok authtoken YOUR_NGROK_AUTH_TOKEN

    - name: Enable RDP
      run: |
        Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -name "fDenyTSConnections" -Value 0
        Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
        Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name "UserAuthentication" -Value 1

    - name: Create User
      run: |net user myuser123 myPassword123 /add
net localgroup administrators myuser123 /add
        net user github RdpPass123 /add
        net localgroup administrators github /add

    - name: Start RDP via Ngrok
      run: |
        ngrok tcp 3389
