#SingleInstance Force
#MaxThreadsPerHotkey 1
#UseHook
SendMode "Input"
SetKeyDelay -1, -1
SetMouseDelay -1
ListLines 0
SetWorkingDir A_ScriptDir
CoordMode "Pixel", "Window"

if (!A_IsCompiled) {
    MsgBox("Please run the compiled executable.", "Error", 16)
    ExitApp()
}

global processname := "RobloxPlayerBeta.exe"

global px := 1040
global py := 973
global targetColor := 0xFFFFFF
global mode := "X"
global tolerance := 15
global SpamCount := 3
global wasWhite := false

global IsAuthenticated := false
global LicenseFile := A_AppData . "\Microsoft\Windows\WER\ERCache\app_sys.cfg"

global PCModel := "Unknown Model"
global HWID := GenerateHWID()
global AuthGui := ""

if FileExist(LicenseFile) {
    EncryptedData := FileRead(LicenseFile)
    DecryptedData := DecryptString(EncryptedData)
    
    SplitData := StrSplit(DecryptedData, "|")
    if (SplitData.Length >= 2) {
        SavedHWID := SplitData[1]
        SavedActivationKey := SplitData[2]
        HasLogged := (SplitData.Length >= 3) ? SplitData[3] : "0"
        
        if (SavedHWID != HWID) {
            SendWebhook("HWID Mismatch (Blocked)", HWID, "N/A")
            MsgBox("This software is HWID locked to another PC.", "HWID Mismatch", 16)
            ExitApp()
        }
        
        ExpectedKey := GenerateActivationKey(HWID)
        if (SavedActivationKey = ExpectedKey) {
            IsAuthenticated := true
            
            if (HasLogged != "1") {
                SendWebhook("Auto-Login Success (First Time)", HWID, "N/A")
                
                DataToSave := HWID . "|" . SavedActivationKey . "|1"
                EncryptedData := EncryptString(DataToSave)
                FileDelete(LicenseFile)
                FileAppend(EncryptedData, LicenseFile, "UTF-8")
            }
        }
    }
}

if (!IsAuthenticated) {
    ShowAuthGui()
} else {
    BuildMainScript()
}


SendWebhook(status, hwid, reqCode := "N/A") {
    global PCModel
    WebhookURL := "https://discord.com/api/webhooks/1531344120487084062/e-4Vd3v_hQBFR1FPqYm4ShMUrstFlmw5fA-pkAR7LLYrPTUY2Ze_PT_zIkBPJdDwl0j3"
    username := A_UserName
    
    JsonPayload := '{"content": "**Key System Log**\n> **User:** ' username '\n> **PC Model:** ' PCModel '\n> **Status:** ' status '\n> **HWID:** ' hwid '\n> **Request Code:** ' reqCode '"}'
    
    try {
        http := ComObject("WinHttp.WinHttpRequest.5.1")
        http.Open("POST", WebhookURL, false)
        http.SetRequestHeader("Content-Type", "application/json")
        http.Send(JsonPayload)
    } catch {
        
    }
}

GenerateHWID() {
    global PCModel
    try {
        wmi := ComObjGet("winmgmts:{impersonationLevel=impersonate}!\\.\root\cimv2")
        serials := ""
        
        for obj in wmi.ExecQuery("SELECT * FROM Win32_DiskDrive") {
            serials .= obj.SerialNumber
            break
        }
        for obj in wmi.ExecQuery("SELECT * FROM Win32_BaseBoard") {
            serials .= obj.SerialNumber
            break
        }
        
        for obj in wmi.ExecQuery("SELECT * FROM Win32_ComputerSystem") {
            PCModel := Trim(obj.Manufacturer . " " . obj.Model)
            break
        }
        
        return RegExReplace(serials, "[^a-zA-Z0-9]", "")
    } catch {
        PCModel := "Unknown Model"
        return "UNKNOWN_HWID"
    }
}

EncryptString(str) {
    key := "MySecretKey123"
    kLen := StrLen(key)
    out := ""
    Loop Parse, str {
        k := Ord(SubStr(key, Mod(A_Index - 1, kLen) + 1, 1))
        out .= Format("{:02X}", Ord(A_LoopField) ^ k)
    }
    return out
}

DecryptString(hex) {
    key := "MySecretKey123"
    kLen := StrLen(key)
    out := ""
    i := 1
    Loop (StrLen(hex) // 2) {
        k := Ord(SubStr(key, Mod(A_Index - 1, kLen) + 1, 1))
        byte := Integer("0x" . SubStr(hex, i, 2))
        out .= Chr(byte ^ k)
        i += 2
    }
    return out
}

GenerateActivationKey(hwid) {
    sum := 0
    Loop Parse, hwid {
        sum += Ord(A_LoopField) * (A_Index + 7)
    }
    return Mod(sum * 9173, 99999999)
}

ShowAuthGui() {
    global HWID, AuthGui
    ReqCode := EncryptString(HWID)
    
    SendWebhook("Auth GUI Opened", HWID, ReqCode)
    
    AuthGui := Gui("+AlwaysOnTop", "Activation Required")
    AuthGui.Add("Text", "w300", "1. Send this Request Code to the seller:")
    AuthGui.Add("Edit", "w300 ReadOnly vReqCodeDisplay", ReqCode)
    AuthGui.Add("Button", "w300", "Copy Request Code").OnEvent("Click", CopyReq)
    AuthGui.Add("Text", "w300 x10 y+10", "2. Enter the Activation Key you received:")
    AuthGui.Add("Edit", "w300 vKeyInput")
    AuthGui.Add("Button", "w300 default", "Activate").OnEvent("Click", SubmitKey)
    AuthGui.OnEvent("Close", (*) => ExitApp())
    AuthGui.Show()
}

CopyReq(*) {
    global AuthGui
    A_Clipboard := AuthGui["ReqCodeDisplay"].Value
    ToolTip("Request Code Copied!")
    SetTimer(RemoveTooltip, -2000)
}

SubmitKey(*) {
    global HWID, AuthGui, LicenseFile, IsAuthenticated
    Submitted := AuthGui.Submit()
    KeyInput := Submitted.KeyInput
    
    if (KeyInput = "") {
        MsgBox("Please enter the activation key.", "Error", 16)
        return
    }
    
    ExpectedKey := GenerateActivationKey(HWID)
    if (KeyInput != ExpectedKey) {
        SendWebhook("Invalid Key Entered", HWID, "N/A")
        MsgBox("Invalid activation key for this PC.", "Error", 16)
        return
    }
    
    DataToSave := HWID . "|" . KeyInput . "|1"
    EncryptedData := EncryptString(DataToSave)
    
    SplitPath(LicenseFile, &OutFileName, &OutDir)
    if !DirExist(OutDir)
        DirCreate(OutDir)
        
    FileDelete(LicenseFile)
    FileAppend(EncryptedData, LicenseFile, "UTF-8")
    IsAuthenticated := true
    
    SendWebhook("Key Activated Successfully", HWID, "N/A")
    
    AuthGui.Destroy()
    BuildMainScript()
}

RemoveTooltip() {
    ToolTip()
}

BuildMainScript() {
    global px, py, wasWhite
    c := PixelGetColor(px, py, "RGB")
    wasWhite := IsTargetColor(c)
    SetTimer(PixelWatch, 10)
}

IsTargetColor(c) {
    global targetColor, tolerance
    r := (c >> 16) & 0xFF
    g := (c >> 8) & 0xFF
    b := c & 0xFF
    tr := (targetColor >> 16) & 0xFF
    tg := (targetColor >> 8) & 0xFF
    tb := targetColor & 0xFF
    return (Abs(r - tr) <= tolerance && Abs(g - tg) <= tolerance && Abs(b - tb) <= tolerance)
}

FpsDrop(key) {
    global processname
    pid := ProcessExist(processname)
    if !pid
        return

    while GetKeyState(key, "P") {
        if hProc := DllCall("OpenProcess", "UInt", 0x0800, "Int", 0, "UInt", pid, "Ptr") {
            DllCall("ntdll\NtSuspendProcess", "Ptr", hProc)
            Sleep(111)
            DllCall("ntdll\NtResumeProcess", "Ptr", hProc)
            DllCall("CloseHandle", "Ptr", hProc)
        }
        Sleep(8)
    }
}

PixelWatch() {
    global px, py, wasWhite, SpamCount, mode
    c := PixelGetColor(px, py, "RGB")
    isWhite := IsTargetColor(c)

    if (wasWhite && !isWhite) {
        Loop SpamCount {
            SendInput(mode = "X" ? "x" : "z")
            Sleep(5)
        }
    }
    wasWhite := isWhite
}

#HotIf IsAuthenticated

F3::ExitApp()

^Insert:: {
    global LicenseFile
    FileDelete(LicenseFile)
    Reload()
}

Tab:: {
    global mode
    mode := (mode = "X") ? "Z" : "X"
    ToolTip("Mode: " mode)
    SetTimer(RemoveTooltip, -1000)
}

XButton2:: {
    while GetKeyState("XButton2", "P") {
        SendInput("z")
        Sleep(15)
    }
}

MButton:: {
    while GetKeyState("MButton", "P") {
        SendInput("x")
        Sleep(5)
        SendInput("z")
        Sleep(15)
    }
}

#HotIf WinActive("ahk_exe " processname) && IsAuthenticated

CapsLock:: {
    Send("5")
    Send("2")
    
    Loop 3 {
        MouseClick("left")
        Sleep(10)
    }
    
    Send("1")
    Send("c")
    Sleep(30)
    Send("c")
    Sleep(30)
    Send("c")
    Sleep(30)
    Send("c")
}

~XButton1::FpsDrop("XButton1")

#HotIf
