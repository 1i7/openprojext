_**Note: this project is not intended to be a fork of original OpenProj project in no way - I would be glad, if the small piece of code added here would go to upstream (free or commercial version of OpenProj), so I will not have to support it anymore**_

_**Note1: for export to SVG check this post by Eduardo Suarez-Santana http://e.suarezsantana.com/content/export-svg-openproj**_

_**Note2: Also see Angelfalls https://github.com/alminium/angelfalls project by Takashi Okamoto which is already a real fork of OpenProj, which contains export to SVG, eport to PNG and much more other features**_


This patch to OpenProj (http://openproj.org) allows **exporting gantt and network diagrams to png image**. It replaces the _"File"/"Export PDF"_ menu item (which actually does not work in the free OpenProj edition and suggests to try the Project On Demand commercial version trial to be able to export your dialgrams to PDF) with _"File"/"Export PNG"_ which exports the whole current diagram to one big PNG image. This might be useful, when you want to show diagrams created in openproj on the computer where OpenProj is not installed (or paste it to some kind of presentation or something).

It does not work as plugin for OpenProj - you will have to replace original **openproj.jar** with the modified one (http://openprojext.googlecode.com/files/openproj.jar-1.4-mod.zip).

In my OpenSUSE installation original **openproj.jar** is located in _"/usr/share/openproj"_, on Windows I saw it somewhere near _"Program Files\Projity\OpenProj"_ - just replace it with the modified one and run application as usual.


## Changelog: ##
- 26 june 2009: recreated openproj.jar binary from OpenProj v1.4 again in hope to solve the [Can't Save to XML](http://code.google.com/p/openprojext/issues/detail?id=2) issue. Also put openproj.jar-1.4-orig.zip, build from unmodified openproj source code, so one can check if some problem is caused by my modifications, or exists in the original source code.

- 9 april 2009: openproj.jar binary in download section updated to OpenProj v1.4 (thank's to susan.or.lee)




Below code modification instructions are based on OpenProj v1.2 (but most likely would be the same for later OpenProj versions)


## Screenshot in action: ##

![http://openprojext.googlecode.com/files/openproj_export_cut.png](http://openprojext.googlecode.com/files/openproj_export_cut.png)



## Download Summary ##
The whole modification consists of 5 modified files (4 of them are mandatory, and the 5th is the Russian lang file) - all located in _"openproj\_ui"_ project. You can download:
  * the [openproj.jar-1.4-mod.zip](http://openprojext.googlecode.com/files/openproj.jar-1.4-mod.zip) - zipped compiled openproj.jar file ready to replace the original openproj.jar, or
  * [modified source files](http://openprojext.googlecode.com/files/openproj-mod-src.tar.gz), or
  * [compiled modified class files](http://openprojext.googlecode.com/files/openproj-mod-bin.tar.gz)


Note, that only the modified parts (which are really not big) of listed files are available under BSD license - all other code belongs to the Projity corporation.


## Prerequirements ##
Download original OpenProj source code archive from [sourceforge page](http://sourceforge.net/project/showfiles.php?group_id=199315&package_id=241282), unpack archive to some folder - the list of directories would be the following:

openproj\_build
openproj\_contrib
openproj\_core
openproj\_reports
openproj\_ui

Make the modifications described below, then go to openproj\_build directory and run 'ant' command - the build process would start

Once it is finished, the output files (openproj.jar and other libraries) can be found in openproj\_build/dist directory

## Modifications Summary ##

The major working code is located in _com.projity.print.GraphPageable.java_

openproj\_ui/src/com/projity/print/GraphPageable.java

add new method printToFile() to this class:

```
	public void printToFile() {
                // Print to png image
                javax.swing.JFileChooser fc = new javax.swing.JFileChooser();
                int returnVal = fc.showSaveDialog(null);
                if (returnVal == javax.swing.JFileChooser.APPROVE_OPTION) {
                        java.io.File file = fc.getSelectedFile();
                        if (!file.getName().toLowerCase().endsWith(".png")) {
                                // append ".png" extension
                                file = new java.io.File(file.getParent(), file.getName() + ".png");
                        }

                        SVGRenderer vr = renderer.createSafePrintCopy();

                        int width = vr.getCanvasSize().width;
                        int height = vr.getCanvasSize().height;
                        java.awt.image.BufferedImage bi = new java.awt.image.BufferedImage(width, height,
                                        java.awt.image.BufferedImage.TYPE_INT_RGB);

                        java.awt.Graphics2D g = bi.createGraphics();
                        g.fillRect(0, 0, width, height);
                        vr.paint(g);

                        try {
                                javax.imageio.ImageIO.write(bi, "PNG", file);
                                // ImageIO.write(bi, "PNG", new File("/home/benderamp/Desktop/project1.png"));
                                // ImageIO.write(bi, "JPEG", new File("/home/benderamp/Desktop/project1.jpg"));
                        } catch (java.io.IOException ex) {
                                ex.printStackTrace();
                        }
                }
        }
```

This block is called from _com.projity.pm.graphic.frames.GraphicManager.java_

openproj\_ui/src/com/projity/pm/graphic/frames/GraphicManager.java

add new printToFile() method to this class:

```
	void printToFile() {
		GraphPageable document=PrintDocumentFactory.getInstance().createDocument(getCurrentFrame(),false);
		if (document!=null) document.printToFile();
	}
```

And this block is also called from the same _GraphicManager.java_ - from the modified PDFAction (the better way could be to add new PNGAction and register in in the main menu, but I did not want to go deeper in the OpenProj code to add new action - replacing existing PDFAction were enough for me):

modify actionPerformed() event handler method in PDFAction inner class inside GraphicManager - the main idea is just to replace savePDF() call to printToFile() call

so it would look like the following:

```

	public class PDFAction extends MenuActionsMap.DocumentMenuAction {
		private static final long serialVersionUID = 1L;
		public void actionPerformed(ActionEvent arg0) {
			setMeAsLastGraphicManager();
			if (isDocumentActive()) {
				Component c = (Component)arg0.getSource();
				Cursor cur = c.getCursor();
				c.setCursor(Cursor.getPredefinedCursor(Cursor.WAIT_CURSOR));
				//savePDF();
				printToFile();
				c.setCursor(cur);
			}
		}
	}
```


Actually, that's almost it - the save-to-png dialog should already pop up instead of print to pdf dialog.


But here a bit more changes required to finish this up:



Rename the menu item in the _com.projity.menu.menu.properties_

openproj\_ui/src/com/projity/menu/menu.properties

just change PDF.text value from "PDF" to "export PNG"

```
PDF.type        = ITEM
PDF.text        = export PNG
PDF.icon        = menu.PDF
PDF.mnemonic    = P
PDF.action      = PDFAction
```


And also there is a small fix in the _com.projity.pm.graphic.spreadsheet.SpreadSheetRenderer.java_ - without it table contents would be invisible in the exported image (though table header and diagrams would be drawn ok)

openproj\_ui/src/com/projity/pm/graphic/spreadsheet/SpreadSheetRenderer.java

find method paintRow() in it, then find a place where different 'component' object's properties are set and add

```
	protected void paintRow(Graphics2D g2, int row, int row0, int h,int col0,int col1,GraphicNode node,Rectangle spreadsheetBounds){

		...

		    	boolean opaque=component.isOpaque();
		    	//component.setDoubleBuffered(false);
		    	component.setOpaque(false);

			// ***** start fix for correct png export
		    	component.setForeground(java.awt.Color.BLACK);
			// ***** end fix for correct png export

				component.setSize(cwidth, params.getConfiguration().getColumnHeaderHeight());
		    	g2.translate(w,h);
		    	component.doLayout();
		    	//g2.setClip(0, 0, cwidth, params.getConfiguration().getColumnHeaderHeight());
		    	component.print(g2);

                ...

	}
```