# c++ ui

### mfc background image

```c++
BOOL CMFCApplication1Dlg::OnInitDialog()
{
	CDialogEx::OnInitDialog();

	// 设置此对话框的图标。  当应用程序主窗口不是对话框时，框架将自动
	//  执行此操作
	SetIcon(m_hIcon, TRUE);			// 设置大图标
	SetIcon(m_hIcon, FALSE);		// 设置小图标

	// TODO: 在此添加额外的初始化代码
	CString strBmpPath = _T(".\\res\\bg.bmp");
	CImage img;
	img.Load(strBmpPath);
	MoveWindow(0, 0, img.GetWidth(), img.GetHeight());
	CBitmap bmpTmp;
	bmpTmp.Attach(img.Detach());
	m_brush.CreatePatternBrush(&bmpTmp);
	return TRUE;  // 除非将焦点设置到控件，否则返回 TRUE
}
HBRUSH CMFCApplication1Dlg::OnCtlColor(CDC* pDC, CWnd* pWnd, UINT nCtlColor)
{
	HBRUSH hbr = CDialogEx::OnCtlColor(pDC, pWnd, nCtlColor);

	// TODO:  在此更改 DC 的任何特性

	// TODO:  如果默认的不是所需画笔，则返回另一个画笔
	return m_brush;
}
```

### qt button 三态

新建qrc文件，添加前缀，添加文件

button右键，改变样式表，添加以下内容

```css
QPushButton{border-image: url(:/new/prefix1/E:/wvv/Desktop/1.png)}
QPushButton:hover{border-image: url(:/new/prefix1/E:/wvv/Desktop/2.png)}
QPushButton:pressed{border-image: url(:/new/prefix1/E:/wvv/Desktop/3.png)}
```

### qt 取消标题栏

```c++
MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    this->setWindowFlag(Qt::FramelessWindowHint);
}
```

### opencv c++工程配置

关键点

属性

-VC++目录-包含目录-build/include;build/include/opencv2;

-库目录-build/x64/vc15/lib

-链接器-输入-附加依赖项-opencv_worldxxx.lib;opencvworldxxxd.lib

系统环境变量

build/x64/vc15/bin

### opencv mat 转 cimage

```c++
int Mat2CImage(Mat* mat, CImage& img) {
	if (!mat || mat->empty())
		return -1;
	int nBPP = mat->channels() * 8;
	img.Create(mat->cols, mat->rows, nBPP);
	if (nBPP == 8)
	{
		static RGBQUAD pRGB[256];
		for (int i = 0; i < 256; i++)
			pRGB[i].rgbBlue = pRGB[i].rgbGreen = pRGB[i].rgbRed = i;
		img.SetColorTable(0, 256, pRGB);
	}
	uchar* psrc = mat->data;
	uchar* pdst = (uchar*)img.GetBits();
	int imgPitch = img.GetPitch();
	for (int y = 0; y < mat->rows; y++)
	{
		memcpy(pdst, psrc, mat->cols * mat->channels());//mat->step is incorrect for those images created by roi (sub-images!)
		psrc += mat->step;
		pdst += imgPitch;
	}
	return 0;
}
```

### mfc opencv 读取摄像头

1. 添加成员变量 VideoCapture *cap;
2. 初始化时，OnInitDialog中，初始化摄像头，启动定时器

```c++
	cap = new VideoCapture(0);
	SetTimer(1, 33, NULL);
```

3. 添加picture control，添加ontimer相应函数

```c++
void CMFCApplication1Dlg::OnTimer(UINT_PTR nIDEvent)
{
	// TODO: 在此添加消息处理程序代码和/或调用默认值
	Mat frame;
	*cap >> frame;
	CImage img;
	Mat2CImage(&frame, img);
	pic.SetBitmap(HBITMAP(img));
	CDialogEx::OnTimer(nIDEvent);
}
```

