---
title: How to add QR code on pdf using java
cover: /images/fsqs-tips-tricks-notes.png
date: 2021-09-14T10:08:22.921Z
categories:
  - QR Code
tags:
  - zxing
  - qr-code
  - java
---

How to add QR code on pdf using java?

<!-- more -->

# Dependencies

```
implementation 'com.google.zxing:core:3.4.1'
implementation 'com.google.zxing:javase:3.4.1'
implementation 'com.itextpdf:itextpdf:5.5.13.2'
```

## Add QR Code on PDF using Java

```java
public class PdfUtils {

    /*
        Add QR code to pdf bytes            
     */
    public static byte[] addQRCode(byte[] pdfBytes, String barcodeText, QRCodePosition position) {
        try (ByteArrayOutputStream os = new ByteArrayOutputStream()) {
            PdfReader reader = new PdfReader(pdfBytes);

            PdfStamper stamper = new PdfStamper(reader, os);

            Image image = Image.getInstance(newQRCodeImage(barcodeText));

            for (int i = 1; i <= reader.getNumberOfPages(); i++) {
                PdfContentByte content = stamper.getOverContent(i);
                image.setAbsolutePosition(position.getAbsoluteX(), position.getAbsoluteY());
                content.addImage(image);
            }

            stamper.close();
            reader.close();
            return os.toByteArray();
        } catch (DocumentException | IOException e) {
            throw new RuntimeException("Error on writing QR code", e);
        }
    }

    /*
        Get new QR code image bytes for barcode text            
    */
    private static byte[] newQRCodeImage(String barcodeText) {
        try (ByteArrayOutputStream image = new ByteArrayOutputStream()) {
            QRCodeWriter barcodeWriter = new QRCodeWriter();
            BitMatrix bitMatrix = barcodeWriter.encode(barcodeText, BarcodeFormat.QR_CODE, 120, 120);
            MatrixToImageWriter.writeToStream(bitMatrix, "png", image);
            return image.toByteArray();
        } catch (WriterException | IOException e) {
            throw new RuntimeException("Error on generating QR code", e);
        }
    }
    
    public enum QRCodePosition {
        TOP_LEFT(0f, 700f),
        TOP_RIGHT(500f, 700f),
        BOTTOM_LEFT(0f, 0f),
        BOTTOM_RIGHT(500f, 0f);

        private final float absoluteX;
        private final float absoluteY;

        QRCodePosition(float absoluteX, float absoluteY) {
            this.absoluteX = absoluteX;
            this.absoluteY = absoluteY;
        }
        
        public float getAbsoluteX() {
            return absoluteX;
        }

        public float getAbsoluteY() {
            return absoluteY;
        }
    }
}

```
