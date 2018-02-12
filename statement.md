# Convert Scanned PDF to OCR (Textsearchable PDF) using C#

![Scanned PDF to OCR](http://www.dotnetspider.com/attachments/Resources/46024-5346902-process.jpg)

## Introduction
Are you looking for a way to convert scanned PDF to Textsearchable PDF ? then read this article, I have explained How to convert Scanned PDF to OCR (Textsearchable PDF) using C# and with the help of some addon tools

Many times we need to scan some files and use them, but as it is scanned and converted to picture format, we can not copy content of the file and it is of no use, so we need some technique which will convert that scanned image to some Text searchable document that can be copied easily,

In such cases we need OCR to convert image in to text. **Optical Character Recognition, or OCR,** is a technology that enables you to convert different types of documents, such as scanned paper documents, PDF files or images captured by a digital camera into editable and searchable data.
This C# template lets you get started quickly with a simple one-page playground.

Are you looking for a code that will convert scanned PDF to OCR ? This article will help you more in order to accomplish your task.

## Let's start cooking

To create a tool which will convert scanned PDF to OCR we need following things.

Things need to collect

1. Ghost script
2. iTextSharp
3. tesseract-ocr
4. C#/ASP.NET (.NET framework 4 and above), Visual stud

**GhostScript :**
It is an interpreter for the PostScript language and for PDF. Ghostscript consists of a PostScript interpreter layer, and a graphics library. Sometimes the Ghostscript graphics library is confusingly also referred to simply as Ghostscript. Even more confusingly, sometimes people say Ghostscript when they really mean GhostPDL. The Ghost script can be download from here : 
[Ghost script Doownload](http://ghostscript.com/download/gsdnld.html)

**ItextSharp :**
iText is a PDF library that allows you to CREATE, ADAPT, INSPECT and MAINTAIN documents in the Portable Document Format (PDF), it can download from here : 
[iTextSharp Download](http://sourceforge.net/projects/itextsharp/)

**Tesseract :**
Tesseract is probably the most accurate open source OCR engine available. Combined with the Image processing library it can read a wide variety of image formats and convert them to text in over 60 languages, you can download it from here :
[Tesseract](http://code.google.com/p/tesseract-ocr/)

With the help of all above components we are able to create scanned PDF to Text searchable PDF


## Digging the code
The code will flow in following direction

First Input Scanned PDF -> using GhostScript get image scanned PDF (Page by Page) -> Run HOCR command on each extracted image using tessract to create .hocr file -> save output file as HTML -> convert the HTML to PDF using iTextSharp PDF Writer
first here we need to take input as scanned file and run ghost script on it, to take out scanned images from PDF file and write it in separate file using ItextSharp

see below code snippet, to know how to get image from scanned file (Page by Page)

```javascript
public string ConvertPDFToBitmap(string PDF, int StartPageNum, int EndPageNum)
        {
            string OutPut = getOutPutFileName(".bmp");
            PDF = "\"" + PDF + "\"";
            string command = String.Concat("-dNOPAUSE -q -r300 -sDEVICE=bmp16m -dBATCH -dFirstPage=", StartPageNum.ToString(), " -dLastPage=", EndPageNum.ToString(), " -sOutputFile=" + OutPut + " " + PDF + " -c quit");  //command to fire with the help of GScript to get image from PDF file
            Process p = new Process ();
            string os = "C:\\Program files\\gs\\gs9.14\\bin\\gswin32c.exe"; //change your ghost script installation path here
            ProcessStartInfo s = new ProcessStartInfo (os, command);
            s.RedirectStandardOutput = true;
            s.RedirectStandardError = true;
            s.CreateNoWindow = true;
            s.UseShellExecute = false;
            p.StartInfo = s;
            p.Start ();
            p.WaitForExit ();
            GC.Collect ();
            return new FileInfo(OutPut.Replace('"', ' ').Trim()).FullName;
        }
```

we can convert image to .hocr.html file using Tesseract or cuneiform, Here we have used Tesseract to create .hocr.html file. See below code snippet, to know how to convert image to .ocr.html file

```javascript
public static string CreateHOCR(OcrMode Mode, string Language, string imagePath)
        {
            string outputFile = imagePath.Replace(Path.GetExtension(imagePath), ".hocr");
            string inputFile = string.Concat('"', imagePath, '"');
            string commandArgs = string.Empty; // Mode == OcrMode.Tesseract ? " -l " + Language + " hocr" : " -l " + Language + " -f hocr -o ";
            string processName = Mode == OcrMode.Tesseract || Mode == OcrMode.TesseractDigitsOnly ? "tesseract" : Mode == OcrMode.Cuneiform ? "cuneiform" : "ocropus-hocr";

            if (Mode == OcrMode.Tesseract)
            {
                string oArg = '"' + outputFile + '"';
                commandArgs = String.Concat(inputFile, " ", oArg, " -l " + Language + " -psm 1 hocr ");
                Process p = new Process();
         string test = string.Concat(processName, " ", commandArgs);
             ProcessStartInfo s = new ProcessStartInfo(processName, commandArgs);
             s.WindowStyle = ProcessWindowStyle.Hidden;
         s.CreateNoWindow = true;
             s.UseShellExecute = true;
             p.StartInfo = s;
                s.WorkingDirectory = @"C:\\Program Files\\Tesseract-OCR\\"; //@"C:\Program Files\Tesseract-OCR\";
                p.Start();
             p.WaitForExit();
         GC.Collect();
            }

            return outputFile + ".html";
        }
```

finally we need to convert .hoct.html file back to pdf (which is our final output), we use iTextSharp PDf write to write content from .hocr.html file to PDF
see below snippet, to know how to write PDF file from .hocr.html

```javascript
private void WriteUnderlayContent(hPage page)
        {
            string pageText = page.Text;
            foreach (hParagraph para in page.Paragraphs)
            {
                foreach (hLine line in para.Lines)
                {
                    if (PDFSettings.WriteTextMode == WriteTextMode.Word)
                    {
                        line.AlignTops();

                        foreach (hWord c in line.Words)
                        {
                            c.CleanText();
                            BBox b = BBox.ConvertBBoxToPoints(c.BBox, PDFSettings.Dpi);

                            if (b.Height > 28)
                                continue;
                            PdfContentByte cb = writer.DirectContentUnder;

                            BaseFont base_font = BaseFont.CreateFont(BaseFont.HELVETICA, BaseFont.WINANSI, false);
                            iTextSharp.text.Font font = new iTextSharp.text.Font(base_font);
                            if (PDFSettings.FontName != null && PDFSettings.FontName != string.Empty)
                            {
                                var fontPath = Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.Fonts), PDFSettings.FontName);
                                base_font = BaseFont.CreateFont(fontPath, BaseFont.IDENTITY_H, BaseFont.EMBEDDED);
                                // BaseFont base_font = BaseFont.CreateFont(BaseFont.HELVETICA, BaseFont.WINANSI, false);
                                font = new iTextSharp.text.Font(base_font);
                            }

                            cb.BeginText();
                            cb.SetFontAndSize(base_font, b.Height > 0 ? b.Height : 2);
                            cb.SetTextMatrix(b.Left, doc.PageSize.Height - b.Top - b.Height + 2);
                            cb.SetWordSpacing(PdfWriter.SPACE);
                            cb.ShowText(c.Text.Trim() + " ");
                            cb.EndText();
                        }
                    }

                }
            }
        }
```

How to run
![How to run](http://www.dotnetspider.com/attachments/Resources/46024-5415278-scr.jpg)

**Prerequisite :** Download Ghostscript, Tesseract from gioven path and then run EXE, Source Code

1. On double click on output exe, you will get following UI.
2. Click on Browse and give input as a scanned folder (A folder with scanned files).
3. Select 'Overide the Files' checkbox, if you want to replace original source file (Here your source PDF files will get replaced by output OCR files).
4. Click on 'Convert to OCR' button to start the process.
5. Cancel to terminate the process.
6. It will create Conversion Report.html file as summary report.
7. You can check output files in 'Ocr_ScanFile' directory on same location of exe.

### Special thanks and references

- https://hocrtopdf.codeplex.com/
- http://code.google.com/p/tesseract-ocr/
- http://sourceforge.net/projects/itextsharp/
- http://soft.rubypdf.com/software/windows-version-jbig2-encoder-jbig2-exe
- http://htmlagilitypack.codeplex.com/
- http://itextpdf.com/
- http://www.ghostscript.com/download/gsdnld.html

I have attached source code and EXE with this article

### Summing Up

With the help of GhostScript, tesseract and iTextsharp, we can create a scanned PDF to textsearchable PDF, a lot can happen with the help of iTextsharp Dlls we can see them in upcoming articles.

Suggestions and Queries always welcome

###Source code

I have attached source code on below link
Source Code(https://drive.google.com/open?id=15U9CVVrs6TYXdAsW11KQbMxn_Ocjm24t)

Thanks
Prasad
