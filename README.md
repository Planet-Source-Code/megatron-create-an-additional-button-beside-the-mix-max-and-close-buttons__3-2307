<div align="center">

## Create an additional button beside the Mix, max and close buttons


</div>

### Description

This code will create an additional button beside the Min, Max and Close buttons! The shape, colours, and image can all be changed to suit your own style.
 
### More Info
 


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[Megatron](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/megatron.md)
**Level**          |Advanced
**User Rating**    |4.0 (16 globes from 4 users)
**Compatibility**  |C\+\+ \(general\), Microsoft Visual C\+\+, Borland C\+\+
**Category**       |[Controls/ Forms/ Dialogs/ Menus](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/controls-forms-dialogs-menus__3-3.md)
**World**          |[C / C\+\+](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/c-c.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/megatron-create-an-additional-button-beside-the-mix-max-and-close-buttons__3-2307/archive/master.zip)

### API Declarations

See code below.


### Source Code

```
#include <windows.h>
const CBX_WHITE = RGB(255,255,255);
const CBX_BLACK = RGB(0,0,0);
const CBX_DKGRAY = RGB(128,128,128);
const CBX_LTGRAY = RGB(224,224,224);
const CBX_GRAY = RGB(192,192,192);
const WM_TASKBOX = (WM_USER+100);
const DCB_DEFAULT = 1;
const DCB_PUSHED = 2;
LRESULT CALLBACK WinProc(HWND, UINT, WPARAM, LPARAM);
void DrawControlBox(HWND, COLORREF, COLORREF, COLORREF, COLORREF, COLORREF, int, DWORD dwFalgs);
void MakeClientPoints(HWND, DWORD, POINT*);
bool GetBoxDimensions(HWND, RECT*);
LPCTSTR lpClass = "TestClass";
HWND hWnd;
HINSTANCE hInstance;
void iASSERT(long num) {
	LPSTR buffer;
	wsprintf(buffer, "%d", num);
	MessageBox(NULL, buffer, "ASSERT", MB_OK);
}
int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInst, LPSTR lpCmdLine, int nShowCmd)
{
	MSG msg;
	WNDCLASSEX wcx;
	HINSTANCE hComCtl = LoadLibrary("Comctl32.lib");
	wcx.cbSize = sizeof(WNDCLASSEX);
	wcx.hInstance = hInstance;
	wcx.lpszClassName = lpClass;
	wcx.lpfnWndProc = WinProc;
	wcx.style = 0;
	wcx.hIcon = LoadIcon(NULL, IDI_APPLICATION);
	wcx.hIconSm = LoadIcon(NULL, IDI_WINLOGO);
	wcx.hCursor = LoadCursor(NULL, IDC_ARROW);
	wcx.lpszMenuName = NULL;
	wcx.cbClsExtra = 0;
	wcx.cbWndExtra = 0;
	wcx.hbrBackground = (HBRUSH)GetStockObject(LTGRAY_BRUSH);
	if(!RegisterClassEx(&wcx)) return 0;
	hWnd = CreateWindow(lpClass, "Test Window", WS_OVERLAPPEDWINDOW, 100,
						100, 400, 400, NULL, NULL, hInstance, NULL );
	ShowWindow(hWnd, nShowCmd);
	SendMessage(hWnd, WM_NCPAINT, NULL, NULL);
	while(GetMessage(&msg, NULL, 0, 0)) {
		TranslateMessage(&msg);
		DispatchMessage(&msg);
	}
	FreeLibrary((HMODULE)hComCtl);
	UnregisterClass(lpClass, hInstance);
	return msg.wParam;
}
//////////////////////////////////////////////////////////////////////////////////////
//		Main Window Callback
//
//		For organization purposes, I've separated the "ControlBox related messages"
//		from the standard messages. If you do not like this, then it can easily be
//		Modified
//////////////////////////////////////////////////////////////////////////////////////
LRESULT CALLBACK WinProc(HWND hWnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
	static bool bPressed;		// Tells when the button is pressed
	POINT PT;
	RECT rcBox;
	if( (uMsg == WM_NCPAINT) || (uMsg == WM_PAINT) || (uMsg == WM_ACTIVATEAPP) || (uMsg == WM_ACTIVATE) ||
		(uMsg == WM_SIZE && wParam != 1) || (uMsg == WM_NCACTIVATE)) {
		DrawControlBox(hWnd, 0, 0, 0, 0, 0, 0, DCB_DEFAULT);
		return DefWindowProc(hWnd, uMsg, wParam, lParam);
	}
	// Change to a pressed image when the left button is pressed
	if( uMsg == WM_NCLBUTTONDOWN ) {
		MakeClientPoints(hWnd, lParam, &PT);
		GetBoxDimensions(hWnd, &rcBox);
		if( (PT.x > rcBox.left-5) && (PT.x < rcBox.right-1) ) {
			bPressed = true;
			DrawControlBox(hWnd, 0, 0, 0, 0, 0, 1, DCB_DEFAULT | DCB_PUSHED);
			return 0;
		}
		return DefWindowProc(hWnd, uMsg, wParam, lParam);
	}
	// Generate a "Click" by sending a custom WM_TASKBOX message.
	if( uMsg == WM_NCLBUTTONUP ) {
		MakeClientPoints(hWnd, lParam, &PT);
		GetBoxDimensions(hWnd, &rcBox);
		DrawControlBox(hWnd, 0, 0, 0, 0, 0, 0, DCB_DEFAULT);
		if( ((PT.x > rcBox.left) && (PT.x < rcBox.right)) && (bPressed == true) ) {
			bPressed = false;
			SendMessage(hWnd, WM_TASKBOX, 0,0);
			return 0;
		}
		bPressed = false;
		return DefWindowProc(hWnd, uMsg, wParam, lParam);
	}
	// Add a pressing effect when the mouse is over the button
	if( (uMsg == WM_NCHITTEST) && GetAsyncKeyState(VK_LBUTTON)) {
		GetBoxDimensions(hWnd, &rcBox);
		MakeClientPoints(hWnd, lParam, &PT);
		if( ((PT.x > rcBox.left-5) && (PT.x < rcBox.right-1)) && ((PT.y > 2-rcBox.bottom) && (PT.y < 2-rcBox.top)))
			DrawControlBox(hWnd, 0, 0, 0, 0, 0, 1, DCB_DEFAULT | DCB_PUSHED);
		else
			DrawControlBox(hWnd, 0, 0, 0, 0, 0, 0, DCB_DEFAULT);
	}
	// Ignore the right clicks because we don't want a system menu
	if (uMsg == WM_NCRBUTTONDOWN) {
		GetBoxDimensions(hWnd, &rcBox);
		MakeClientPoints(hWnd, lParam, &PT);
		if( ((PT.x > rcBox.left-5) && (PT.x < rcBox.right-1)) && ((PT.y > 2-rcBox.bottom) && (PT.y < 2-rcBox.top)))
			return 0;
		return DefWindowProc(hWnd, uMsg, wParam, lParam);
	}
  switch(uMsg) {
		case WM_DESTROY:
			ReleaseCapture();
			PostQuitMessage(0);
			return 0;
		// WM_TASKBOX is out custom message
		case WM_TASKBOX:
			MessageBox(NULL, "You pressed the task box", "", MB_OK);
			return 0;
		default:
			return DefWindowProc(hWnd, uMsg, wParam, lParam);
	}
}
// Converts the coordinates from lParam into a point structure and relative to the client
void MakeClientPoints(HWND hWnd, DWORD dwPoints, POINT *pts) {
	pts->x = LOWORD(dwPoints);
	pts->y = HIWORD(dwPoints);
	ScreenToClient(hWnd, pts);
}
//	Gets the dimensions of the ControlBox
bool GetBoxDimensions(HWND hWnd, RECT* rcBox) {
	RECT rc;
	GetWindowRect(hWnd, &rc);
	DWORD dwStyle = GetWindowLong(hWnd, GWL_STYLE);
	bool bIsMax = false;
	bool bIsMin = false;
	// Check the style to make sure we draw it in the right place
	if( dwStyle & WS_MAXIMIZEBOX )
		bIsMax = true;
	if( dwStyle & WS_MINIMIZEBOX )
		bIsMin = true;
	if( (GetMenuState(GetSystemMenu(hWnd, 0), 4, MF_BYPOSITION) & MF_GRAYED) &&
	  !(GetMenuState(GetSystemMenu(hWnd, 0), 3, MF_BYPOSITION) & MF_GRAYED) )
		bIsMax = true;
	if( (GetMenuState(GetSystemMenu(hWnd, 0), 3, MF_BYPOSITION) & MF_GRAYED) &&
	  !(GetMenuState(GetSystemMenu(hWnd, 0), 4, MF_BYPOSITION) & MF_GRAYED) )
		bIsMin = true;
	if((GetMenuState(GetSystemMenu(hWnd, 0), 3, MF_BYPOSITION) & MF_GRAYED) &&
	  (GetMenuState(GetSystemMenu(hWnd, 0), 4, MF_BYPOSITION) & MF_GRAYED) )
	{
		rcBox->left = (rc.right - rc.left)-((74/2)+2);
		rcBox->right = rcBox->left + 16;
		rcBox->top = 6;
		rcBox->bottom = 20;
		return true;
	}
	if( ((dwStyle & WS_MINIMIZEBOX) ) || ((dwStyle & WS_MAXIMIZEBOX) && bIsMin) ) {
		rcBox->left = (rc.right - rc.left)-74;
		rcBox->right = rcBox->left + 16;
		rcBox->top = 6;
		rcBox->bottom = 20;
		return true;
	}
	return false;
}
// Draws the control box
void DrawControlBox(HWND hWnd,		// Handle of window to draw on
					COLORREF crBg,	// Background Color
					COLORREF Bdm1,	// Bottom border
					COLORREF Bdm2,	// 2nd level bottom border
					COLORREF Top1,	// Top border
					COLORREF Top2,	// 2nd level top border
					int iOffset,	// Amount to offest the ellipse by
					DWORD dwFlags)
{
	HBRUSH hBrush = NULL;
	HBRUSH hOldBrush = NULL;
	HPEN hPen = NULL;
	HPEN hOldPen = NULL;
	COLORREF crEllipse;
	HDC	hDC = GetWindowDC(hWnd);
	POINT PT;
	RECT rcBox;
	crEllipse = RGB(0,0,0);
	if( dwFlags & DCB_DEFAULT ) {
		if( dwFlags & DCB_PUSHED ) {
			crBg = GetSysColor(COLOR_3DFACE);
			Top1 = GetSysColor(COLOR_3DDKSHADOW);
			Top2 = GetSysColor(COLOR_3DSHADOW);
			Bdm1 = GetSysColor(COLOR_3DHILIGHT);
			Bdm2 = GetSysColor(COLOR_3DLIGHT);
			crEllipse = GetSysColor(COLOR_BTNTEXT);
		}
		else {
			crBg = GetSysColor(COLOR_3DFACE);
			Bdm1 = GetSysColor(COLOR_3DDKSHADOW);
			Bdm2 = GetSysColor(COLOR_3DSHADOW);
			Top1 = GetSysColor(COLOR_3DHILIGHT);
			Top2 = GetSysColor(COLOR_3DLIGHT);
			crEllipse = GetSysColor(COLOR_BTNTEXT);
		}
	}
	GetBoxDimensions(hWnd, &rcBox);
	// Draw the rectangle and top border
	hBrush = CreateSolidBrush(crBg);
	hOldBrush = (HBRUSH)SelectObject(hDC, (HGDIOBJ)hBrush);
	hPen = CreatePen(0, 1, Top1);
	hOldPen = (HPEN)SelectObject(hDC, hPen);
	Rectangle(hDC, rcBox.left, rcBox.top, rcBox.right, rcBox.bottom);
	DeleteObject(SelectObject(hDC, hOldPen));
	// Draw the bottom border
	hPen = CreatePen(0,1,Bdm1);
	hOldPen = (HPEN)SelectObject(hDC, hPen);
	MoveToEx(hDC, rcBox.left, rcBox.bottom-1, &PT);
	LineTo(hDC,rcBox.right,rcBox.bottom-1);
	MoveToEx(hDC, rcBox.right-1, rcBox.top, &PT);
	LineTo(hDC, rcBox.right-1, rcBox.bottom-1);
	DeleteObject(SelectObject(hDC,hOldPen));
	// Draw the ellipse (black regardless of parameters)
	hPen = CreatePen(0,1,crEllipse);
	hOldPen = (HPEN)SelectObject(hDC, hPen);
	Ellipse(hDC, rcBox.left+3 +iOffset,rcBox.top+3 +iOffset,rcBox.right-4 +iOffset,rcBox.bottom-3 +iOffset);
	DeleteObject(SelectObject(hDC,hOldPen));
	DeleteObject(SelectObject(hDC,hOldBrush));
	// Draw the 2nd level bottom border
	hPen = CreatePen(0,1,Bdm2);
	hOldPen = (HPEN)SelectObject(hDC, hPen);
	MoveToEx(hDC, rcBox.left+1, rcBox.bottom-2, &PT);
	LineTo(hDC,rcBox.right-1,rcBox.bottom-2);
	MoveToEx(hDC, rcBox.right-2, rcBox.top+1, &PT);
	LineTo(hDC, rcBox.right-2, rcBox.bottom-1);
	DeleteObject(SelectObject(hDC,hOldPen));
	// Draw the 2nd level top border
  hPen = CreatePen(0, 1, Top2);
  hOldPen = (HPEN)SelectObject(hDC, hPen);
	MoveToEx(hDC, rcBox.left+1,rcBox.top+1,&PT);
	LineTo(hDC, rcBox.right-2, rcBox.top+1);
	MoveToEx(hDC, rcBox.left+1,rcBox.top+1,&PT);
	LineTo(hDC, rcBox.left+1,rcBox.bottom-2);
	DeleteObject(SelectObject(hDC, hOldPen));
	ReleaseDC(hWnd, hDC);
}
```

