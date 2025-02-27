# cupsfilter: swap black with dark blue
This hack uses cupsFilter2 to color-exchange black with dark blue color in all printed documents automatically on the fly. It does this by converting PDF to PNG then back to PDF with a hook inside the CUPS printing chain.

It is mainly useful for printers with busted/empty black toner.

Please note that ChatGPT (even o1) currently doesn't understand CUPS and Postscript that well, so it might make a lot of errors if you go into too many details. It was quite a struggle to make it work at all. Hence there is no color-swapping on a Postscript-level but only on a pixel-by-pixel basis in converted images. Be mindful that printing images instead of text causes a fairly large decrease in printing speed.

The script was tested with the universal IPP printer driver, which is an alternative/replacement to the classic custom drivers that you can use with almost all printers and it works just as good. It is highly recommended that you install your printer as IPP for this script.

## Dependencies

imagemagick, ghostscript, python-pillow

## Installation

1. The scripts uses 300 dpi for all pages. Modify the `blacktoblue` script if you want something else.
2. Copy files and change permissions:
   ```
   chmod 755 usr-lib-cups-filter/*
   sudo cp -p usr-lib-cups-filter/* /usr/lib/cups/filter/
   ```
3. Edit the PPD file that cups created for your printer, for example `/etc/cups/ppd/Dell.ppd`:
   Locate a line such as this:
   ```
   *cupsFilter2: "application/vnd.cups-pdf application/pdf 0 -"
   ```
   And replace it with this:
   ```
   *cupsFilter2: "application/vnd.cups-pdf application/pdf 0 /usr/lib/cups/filter/blacktoblue"
   ```
4. **Restart cups!**
   ```
   systemctl restart cups
   ```
## Usage

To test the script and check if you have all dependences and such, you can use it directly:
   ```
   cat test.pdf | /usr/lib/cups/filter/blacktoblue > test_output.pdf
   ```

If black is blue now, try to print test.pdf directly on your printer and black should be blue there as well! (beware that if output is malformed, dozens of garbage pages will be printed).

## Using different colors and color thresholds

I gave the GIMP color exchange code to ChatGPT-o1 to write `blacktoblue_image`. In GIMP color exchange, you can define red/green/blue thresholds to how aggressive the rotation is. It is fairly obvious how to change `blacktoblue_image` to make it swap black with different thresholds or different colors.

## Issues caused by not using IPP but classic driver
If you use a classic driver, you might need to modify your PPD file, and have ChatGPT adapt the `blacktoblue` script to process different input or output formats other than PDF. In the PPD file, as far as I understand it `application/vnd.cups-pdf` is the input stream and `application/pdf` the output format. cupsFilter2 rules will be applied in succession, until the printer driver reports (or misreports like IPP driver, which accepts `*` ...) to support the output format. If you are not using IPP, you might have a cupsFilter2 line that takes vnd.cups-ps as input or outputs Postscript, or something like that.
