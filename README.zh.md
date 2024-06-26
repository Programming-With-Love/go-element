## 用于在Go中进行Windows窗体UI自动化的Uiautomation库
[![Go Reference](https://pkg.go.dev/badge/github.com/auuunya/go-element.svg)](https://pkg.go.dev/github.com/auuunya/go-element)

### 概述
该库为使用UI自动化框架在Windows上自动执行UI方法提供了一个方便的接口。它允许您通过相应的Windows COM接口与应用程序中的UI元素控件进行交互和操作。该库可将Go方法映射到COM接口中定义的方法，从而轻松在Go应用程序中使用UI自动化.

### 特性
- **COM接口映射:** 该库反映了用户界面自动化COM接口的结构，与 Windows API 结构相对应
- **元素结构化:** 可更直观地显示应用程序用户界面元素的结构
- **Automation:** 以编程方式执行用户界面自动化任务，如读取或操作UI控件的状态。

### 安装
```shell
go get -u github.com/auuunya/go-element
```
```go
import (
    ...
    uiautomation "github.com/auuunya/go-element"
)
```

### 示例用法:
**浏览器元素结构输出**
```go
func main() {
	uiautomation.CoInitialize()
	defer uiautomation.CoUninitialize()
	findhwnd := uiautomation.GetWindowForString("Chrome_WidgetWin_1", "")
	instance, _ := uiautomation.CreateInstance(uiautomation.CLSID_CUIAutomation, uiautomation.IID_IUIAutomation, uiautomation.CLSCTX_INPROC_SERVER|uiautomation.CLSCTX_LOCAL_SERVER|uiautomation.CLSCTX_REMOTE_SERVER)
	unk := uiautomation.NewIUnKnown(instance)
	ppv := uiautomation.NewIUIAutomation(unk)
	root, _ := uiautomation.ElementFromHandle(ppv, findhwnd)
	elems := uiautomation.TraverseUIElementTree(ppv, root)
	uiautomation.TreeString(elems, 0)
}
```

**查找记事本可编辑空间并且输入文本**
```go
var (
	dll                 = syscall.NewLazyDLL("User32.dll")
	procSendInput       = dll.NewProc("SendInput")
	procIsWindowEnabled = dll.NewProc("IsWindowEnabled")
	procSendMessageW    = dll.NewProc("SendMessageW")
)

const (
	WM_SETTEXT = 0x000C
	WM_KEYDOWN = 0x0100
	VK_RETURN  = 0x0D
)

func main() {
	uiautomation.CoInitialize()
	defer uiautomation.CoUninitialize()
	findhwnd := uiautomation.GetWindowForString("Notepad", "")
	instance, _ := uiautomation.CreateInstance(uiautomation.CLSID_CUIAutomation, uiautomation.IID_IUIAutomation, uiautomation.CLSCTX_INPROC_SERVER|uiautomation.CLSCTX_LOCAL_SERVER|uiautomation.CLSCTX_REMOTE_SERVER)
	unk := uiautomation.NewIUnKnown(instance)
	ppv := uiautomation.NewIUIAutomation(unk)
	root, _ := uiautomation.ElementFromHandle(ppv, findhwnd)
	elems := uiautomation.TraverseUIElementTree(ppv, root)
	fn := func(elem *uiautomation.Element) bool {
		return elem.CurrentControlType == uiautomation.UIA_DocumentControlTypeId && elem.CurrentName == "RichEditD2DPT"
	}
	foundElement := uiautomation.SearchElem(elems, fn)
	uia_hwnd := foundElement.CurrentNativeWindowHandle
	unk, _ = foundElement.UIAutoElement.GetCurrentPattern(uiautomation.UIA_ValuePatternId)
	text := "Hello World!\nHello UI Automation!!!"
	content, err := windows.UTF16PtrFromString(text)
	if err != nil {
		return
	}
	retSendText, _, _ := procSendMessageW.Call(
		uintptr(uia_hwnd),
		uintptr(WM_SETTEXT),
		0,
		uintptr(unsafe.Pointer(content)),
	)
	fmt.Printf("retSendText: %#v\n", retSendText)
}
```
**搜索文件夹指定的控件**
```go
func main() {
	uiautomation.CoInitialize()
	defer uiautomation.CoUninitialize()
	findhwnd := uiautomation.GetWindowForString("CabinetWClass", "")
	instance, _ := uiautomation.CreateInstance(uiautomation.CLSID_CUIAutomation, uiautomation.IID_IUIAutomation, uiautomation.CLSCTX_INPROC_SERVER|uiautomation.CLSCTX_LOCAL_SERVER|uiautomation.CLSCTX_REMOTE_SERVER)
	unk := uiautomation.NewIUnKnown(instance)
	ppv := uiautomation.NewIUIAutomation(unk)
	root, _ := uiautomation.ElementFromHandle(ppv, findhwnd)
	elems := uiautomation.TraverseUIElementTree(ppv, root)
	fn := func(elem *uiautomation.Element) bool {
		return elem.CurrentClassName == "SelectorButton" && elem.CurrentName == "详细信息"
	}
	foundElement := uiautomation.SearchElem(elems, fn)
	fmt.Printf("foundElement: %v\n", foundElement)
}
```

### 任务列表
- [ ] ui结构写入到文件
- [ ] 更为细化的ui操作

### 贡献
欢迎贡献！如果您发现错误或想添加一些功能，请随时提出问题或提交拉取请求。

### 许可证
该库使用MIT许可证