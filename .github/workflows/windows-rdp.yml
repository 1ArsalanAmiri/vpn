name: Windows Cloud RDP

on:
  workflow_dispatch:

jobs:
  build:
    runs-on: windows-latest
    timeout-minutes: 360

    steps:
      - name: 1. Enable Remote Desktop & Firewall
        run: |
          Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -name "fDenyTSConnections" -Value 0
          Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
          Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name "UserAuthentication" -Value 0

      - name: 2. Create Admin Account
        run: |
          $Password = ConvertTo-SecureString "P@ssw0rd1qazXSW@#" -AsPlainText -Force
          New-LocalUser -Name "NvDAdmin" -Password $Password -PasswordNeverExpires -ErrorAction SilentlyContinue
          Add-LocalGroupMember -Group "Administrators" -Member "NvDAdmin" -ErrorAction SilentlyContinue
          net user runneradmin P@ssw0rd1qazXSW@#

      - name: 3. Install & Connect Tailscale VPN
        env:
          TAILSCALE_AUTHKEY: ${{ secrets.TAILSCALE_AUTHKEY }}
        run: |
          # Download and install Tailscale silently
          Invoke-WebRequest "https://pkgs.tailscale.com/stable/tailscale-setup-latest-amd64.msi" -OutFile "C:\tailscale.msi"
          Start-Process msiexec -ArgumentList "/i C:\tailscale.msi /quiet /norestart" -Wait
          
          # Connect to Tailscale network
          Start-Sleep -Seconds 5
          & "C:\Program Files\Tailscale\tailscale.exe" up --authkey="$Env:TAILSCALE_AUTHKEY" --hostname="github-rdp-server"
          
          Start-Sleep -Seconds 8
          
          # Get Tailscale IP
          $ts_ip = & "C:\Program Files\Tailscale\tailscale.exe" ip -4
          
          Write-Host ""
          Write-Host "==================================================" -ForegroundColor Green
          Write-Host "  >>> RDP SERVER IS READY! <<<" -ForegroundColor Cyan
          Write-Host "  Tailscale IP : $ts_ip" -ForegroundColor Yellow
          Write-Host "  Username     : NvdAdmin" -ForegroundColor Yellow
          Write-Host "  Password     : P@ssw0rd1qazXSW@#" -ForegroundColor Yellow
          Write-Host "  >> Open mstsc and connect to: $ts_ip" -ForegroundColor Magenta
          Write-Host "==================================================" -ForegroundColor Green

      - name: 4. Keep Server Alive (48 Hours)
        run: |
          for ($i = 1; $i -le 2880; $i++) {
            Start-Sleep -Seconds 60
            Write-Host "[$i/2880] Server running..."
          }
