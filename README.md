# agent-pipeline — `ap`

Trang cài đặt `ap` — app một-máy của agent-pipeline. Mỗi GitHub Release ở repo này đính kèm đúng một **Bản phát hành** — zip nguyên cây `dist\` (`ap.exe` + `gui\`) của một phiên bản. Cài ap là dán một khối PowerShell: tải bản mới nhất, giải nén vào **Thư mục cài** cố định, ghi user PATH. Không cần quyền admin, không cần token, không cần `gh`.

## Điều kiện tiên quyết

Windows + `pwsh` 7 + `gh` + `git` + omp + Orca (host của mọi Cửa sổ run). ap không cross-platform. Cài xong chạy `ap doctor` để máy tự khai còn thiếu gì.

## Cài đặt

Mở PowerShell (Windows PowerShell 5.1 hoặc `pwsh` 7 đều được) rồi dán nguyên khối này:

```powershell
# Cài ap: tải Bản phát hành mới nhất của Repo phát hành, giải nén vào Thư mục cài, ghi user PATH.
$repo    = 'ChauThan/agent-pipeline-distributions'
$install = Join-Path $env:LOCALAPPDATA 'Programs\ap'

$release = (Invoke-WebRequest -UseBasicParsing "https://api.github.com/repos/$repo/releases/latest" -Headers @{ 'User-Agent' = 'ap-install' }).Content | ConvertFrom-Json
$asset   = $release.assets | Where-Object { $_.name -like 'ap-*-win-x64.zip' } | Select-Object -First 1
if (-not $asset) { throw "Release $($release.tag_name) không có asset ap-*-win-x64.zip" }

New-Item -ItemType Directory -Force -Path $install | Out-Null
$zip = Join-Path $env:TEMP $asset.name
Invoke-WebRequest -UseBasicParsing $asset.browser_download_url -OutFile $zip
Expand-Archive -Path $zip -DestinationPath $install -Force
Remove-Item $zip -Force

$userPath = [Environment]::GetEnvironmentVariable('Path', 'User')
if (($userPath -split ';') -notcontains $install) {
    $newPath = if ($userPath) { $userPath.TrimEnd(';') + ';' + $install } else { $install }
    [Environment]::SetEnvironmentVariable('Path', $newPath, 'User')
}

Write-Host "Đã cài ap vào $install. Mở terminal mới rồi chạy: ap doctor"
```

Snippet làm bốn việc: hỏi release mới nhất của `ChauThan/agent-pipeline-distributions` qua GitHub API công khai (ẩn danh), tải asset `ap-*-win-x64.zip`, giải nén đè vào Thư mục cài `%LOCALAPPDATA%\Programs\ap` (tạo nếu thiếu), rồi thêm Thư mục cài vào user PATH nếu chưa có. Thuần PowerShell (`Invoke-WebRequest` + `Expand-Archive` + `[Environment]::SetEnvironmentVariable`), không auth, không `gh`, không admin. Lần đầu chạy có thể bị SmartScreen chặn — xem mục dưới.

Mọi thứ sống theo từng người dùng: `ap` nằm dưới `%LOCALAPPDATA%`, vùng data nằm trong `AP_HOME`, không đụng gì cần quyền admin.

## Kiểm tra môi trường

Mở một terminal **mới** (PATH chỉ đổi cho tiến trình khởi động sau), rồi chạy:

```
ap doctor
```

`ap doctor` kiểm `gh`, `git`, `pwsh`, Orca host và PATH của Thư mục cài — in từng mục `OK`/`THIẾU` kèm gợi ý khắc phục, exit code phản ánh có mục nào thiếu. Sửa hết `THIẾU` trước khi chạy issue đầu tiên. `ap version` in phiên bản ap đang chạy khi cần đối chiếu.

## SmartScreen — exe chưa ký

ap chưa ký số, nên lần đầu chạy `ap.exe` Windows có thể hiện hộp thoại "Windows protected your PC" với nhà phát hành "Unknown publisher". Đây là cảnh báo trên file tải về, không phải virus. Ba cách bỏ:

1. Ngay ở hộp thoại: bấm "More info" rồi "Run anyway".
2. Bỏ mark-of-the-web: chuột phải `ap.exe` (và file trong `gui\` nếu bị chặn) → Properties → tab General → tích "Unblock" → OK.
3. Nếu Windows từ chối chạy app ngoài Microsoft Store: Settings → Apps → Advanced app settings → "Choose where to get apps" → Anywhere.

## Nâng cấp

```
ap upgrade
```

`ap upgrade` hỏi Bản phát hành mới nhất rồi thay nội dung Thư mục cài (rename-dance xử lý exe đang chạy). Đã là bản mới nhất thì no-op thành công. Nâng cấp chỉ đụng máy — giữ nguyên `AP_HOME` (Hàng đợi toàn cục, danh sách Đăng ký repo, Nhật ký Runner) lẫn Hợp đồng. Sau khi cài, ap còn tự Kiểm tra bản mới tối đa một lần mỗi ngày và in đúng một dòng nhắc khi có bản mới.

## Tải bản cũ thủ công

Trang Releases: <https://github.com/ChauThan/agent-pipeline-distributions/releases> — chọn một release cũ hơn, tải `ap-<version>-win-x64.zip`, rồi giải nén đè vào Thư mục cài:

```powershell
Expand-Archive -Path .\ap-<version>-win-x64.zip -DestinationPath (Join-Path $env:LOCALAPPDATA 'Programs\ap') -Force
```

Đường này để tự hạ chân khi bản mới hỏng, không cần `gh` lẫn token.
