# 下面是一个PowerShell脚本，旨在令您调用Microsoft Office 365桌面版的相关功能，以批量转换Word文档、PowerPoint演示和Excel表格至PDF文档。

````
# 加载Windows.Forms和Drawing程序集
Add-Type -AssemblyName System.Windows.Forms
Add-Type -AssemblyName System.Drawing

# 创建文件选择对话框函数
function Show-FileDialog {
    param (
        [string]$title,
        [string]$filter
    )
    $fileDialog = New-Object System.Windows.Forms.OpenFileDialog
    $fileDialog.Filter = $filter
    $fileDialog.Title = $title
    $fileDialog.Multiselect = $true
    if ($fileDialog.ShowDialog() -eq [System.Windows.Forms.DialogResult]::OK) {
        return $fileDialog.FileNames
    } else {
        return $null
    }
}

# 使用Shell.Application创建更直观的文件夹选择对话框
function Show-FolderDialog {
    param (
        [string]$description
    )
    $shell = New-Object -ComObject Shell.Application
    $folder = $shell.BrowseForFolder(0, $description, 0)
    if ($folder -ne $null) {
        return $folder.Self.Path
    } else {
        return $null
    }
}

# 定义转换函数：将PowerPoint文件转换为PDF
function Convert-PowerPointToPDF {
    param (
        [string]$inputFile,
        [string]$outputFolder
    )
    try {
        $outputFile = [System.IO.Path]::ChangeExtension("$outputFolder\$([System.IO.Path]::GetFileNameWithoutExtension($inputFile))", "pdf")
        $presentation = $pptApp.Presentations.Open($inputFile, [Microsoft.Office.Core.MsoTriState]::msoFalse, [Microsoft.Office.Core.MsoTriState]::msoFalse, [Microsoft.Office.Core.MsoTriState]::msoFalse)
        $presentation.SaveAs($outputFile, [Microsoft.Office.Interop.PowerPoint.PpSaveAsFileType]::ppSaveAsPDF)
        $presentation.Close()
        [System.Runtime.Interopservices.Marshal]::ReleaseComObject($presentation)
    } catch {
        Write-Host "Error converting $inputFile to PDF: $_"
    }
}

# 定义转换函数：将Word文件转换为PDF
function Convert-WordToPDF {
    param (
        [string]$inputFile,
        [string]$outputFolder
    )
    try {
        $outputFile = [System.IO.Path]::ChangeExtension("$outputFolder\$([System.IO.Path]::GetFileNameWithoutExtension($inputFile))", "pdf")
        $document = $wordApp.Documents.Open($inputFile)
        $document.SaveAs([ref] $outputFile, [ref] [Microsoft.Office.Interop.Word.WdSaveFormat]::wdFormatPDF)
        $document.Close()
        [System.Runtime.Interopservices.Marshal]::ReleaseComObject($document)
    } catch {
        Write-Host "Error converting $inputFile to PDF: $_"
    }
}

# 定义转换函数：将Excel文件转换为PDF
function Convert-ExcelToPDF {
    param (
        [string]$inputFile,
        [string]$outputFolder
    )
    try {
        $outputFile = [System.IO.Path]::ChangeExtension("$outputFolder\$([System.IO.Path]::GetFileNameWithoutExtension($inputFile))", "pdf")
        $workbook = $excelApp.Workbooks.Open($inputFile)
        $workbook.ExportAsFixedFormat([Microsoft.Office.Interop.Excel.XlFixedFormatType]::xlTypePDF, $outputFile)
        $workbook.Close($false)
        [System.Runtime.Interopservices.Marshal]::ReleaseComObject($workbook)
    } catch {
        Write-Host "Error converting $inputFile to PDF: $_"
    }
}

# 选择文件
$filePaths = Show-FileDialog -title "请选择要转换的文件" -filter "所有支持的文件 (*.ppt; *.pptx; *.doc; *.docx; *.xls; *.xlsx)|*.ppt;*.pptx;*.doc;*.docx;*.xls;*.xlsx"
if (-not $filePaths) {
    Write-Host "未选择文件，脚本结束"
    exit
}

# 选择输出文件夹
$outputFolder = Show-FolderDialog -description "请选择输出PDF文件的文件夹"
if (-not $outputFolder) {
    Write-Host "未选择输出文件夹，脚本结束"
    exit
}

# 加载应用程序
$pptApp = $null
$wordApp = $null
$excelApp = $null

foreach ($filePath in $filePaths) {
    # 根据文件扩展名调用相应的转换函数
    $extension = [System.IO.Path]::GetExtension($filePath).ToLower()

    switch ($extension) {
        ".ppt" {
            if (-not $pptApp) {
                # 加载PowerPoint应用程序
                $pptApp = New-Object -ComObject PowerPoint.Application
                $pptApp.Visible = [Microsoft.Office.Core.MsoTriState]::msoTrue
            }
            Convert-PowerPointToPDF $filePath $outputFolder
        }
        ".pptx" {
            if (-not $pptApp) {
                # 加载PowerPoint应用程序
                $pptApp = New-Object -ComObject PowerPoint.Application
                $pptApp.Visible = [Microsoft.Office.Core.MsoTriState]::msoTrue
            }
            Convert-PowerPointToPDF $filePath $outputFolder
        }
        ".doc" {
            if (-not $wordApp) {
                # 加载Word应用程序
                $wordApp = New-Object -ComObject Word.Application
                $wordApp.Visible = $false
            }
            Convert-WordToPDF $filePath $outputFolder
        }
        ".docx" {
            if (-not $wordApp) {
                # 加载Word应用程序
                $wordApp = New-Object -ComObject Word.Application
                $wordApp.Visible = $false
            }
            Convert-WordToPDF $filePath $outputFolder
        }
        ".xls" {
            if (-not $excelApp) {
                # 加载Excel应用程序
                $excelApp = New-Object -ComObject Excel.Application
                $excelApp.Visible = $false
            }
            Convert-ExcelToPDF $filePath $outputFolder
        }
        ".xlsx" {
            if (-not $excelApp) {
                # 加载Excel应用程序
                $excelApp = New-Object -ComObject Excel.Application
                $excelApp.Visible = $false
            }
            Convert-ExcelToPDF $filePath $outputFolder
        }
        default {
            Write-Host "不支持的文件类型：$extension"
        }
    }
}

# 关闭应用程序
if ($pptApp) {
    $pptApp.Quit()
    [System.Runtime.Interopservices.Marshal]::ReleaseComObject($pptApp)
}
if ($wordApp) {
    $wordApp.Quit()
    [System.Runtime.Interopservices.Marshal]::ReleaseComObject($wordApp)
}
if ($excelApp) {
    $excelApp.Quit()
    [System.Runtime.Interopservices.Marshal]::ReleaseComObject($excelApp)
}

Write-Host "文件转换完成，PDF保存在：$outputFolder"
````
