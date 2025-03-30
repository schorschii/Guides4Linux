# Configure Datalogic QR/barcode scanner to send line break (\n) as ENTER 

Datalogic scanners are normally sending CTRL+J when reading a line feed (LF) character and ENTER when reading carriage return (CR).

That's not cool if you use the scanner with Linux or Android since QR codes mostly use Unix line endings (LF only) when they contain line breaks.

You can work around by using the character replacement feature of these scanners.

Use the Datalogic online tool to create a config code: https://aladdin.datalogic.com/

1. Select your scanner
2. Go to "Data Format" -> "Character Replacement"
3. Enter "0A0D" for "Character Conversion" to tell the scanner to replace LF (hex "0A") with CR (hex "0D")
4. Click "Export Configuration" and scan the code
