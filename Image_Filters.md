# Image Filters

## Gaussian Filter

### code

code link: [GaussianFilter](https://github.com/joeyajames/Java/blob/master/Image%20Filters/ImageFilter.java)

```java
import java.util.*;
import java.awt.*;
import java.awt.image.*;
import java.io.*;
import javax.imageio.ImageIO;
import javax.swing.*;
import java.awt.geom.AffineTransform;

public class ImageFilterGaussian {

	public static void main(String[] args) {
        String path = "C:\\Users\\solso\\Pictures\\oranges.jpg";
		File file = new File(path);
		//File file = new File("License Plate Photos/ca_10.jpeg");
		BufferedImage img = null;

		try { img = ImageIO.read(file); } 
		catch (IOException e) { e.printStackTrace(System.out); }

		if (img != null) {
			display(img);
			//img = brighten(img);
			img = toGrayScale(img);
			//img = toGrayScale2(img);
			display(img);
			//img = pixelate(img);
			img = pixelate2(img, 3);
			//img = resize(img, 150);
			img = blur(img);
			//img = blur(blur(img));
			//img = heavyblur(img);
			//img = detectEdges(img);
			display(img);
		}
	}

	// display image in a JPanel popup
	public static void display (BufferedImage img) {
		System.out.println("  Displaying image.");
		JFrame frame = new JFrame();
	    JLabel label = new JLabel();
		frame.setSize(img.getWidth(), img.getHeight());
		label.setIcon(new ImageIcon(img));
		frame.getContentPane().add(label, BorderLayout.CENTER);
		frame.setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);
		frame.pack();
		frame.setVisible(true);
	}

	// convert image to grayscale
	public static BufferedImage toGrayScale (BufferedImage img) {
		System.out.println("  Converting to GrayScale.");
		BufferedImage grayImage = new BufferedImage(
			img.getWidth(), img.getHeight(), BufferedImage.TYPE_BYTE_GRAY);
		Graphics g = grayImage.getGraphics();
		g.drawImage(img, 0, 0, null);
		g.dispose();
		return grayImage;
	}

	public static BufferedImage toGrayScale2 (BufferedImage img) {
		System.out.println("  Converting to GrayScale2.");
		BufferedImage grayImage = new BufferedImage(
			img.getWidth(), img.getHeight(), BufferedImage.TYPE_BYTE_GRAY);
		int rgb=0, r=0, g=0, b=0;
		for (int y=0; y<img.getHeight(); y++) {
			for (int x=0; x<img.getWidth(); x++) {
				rgb = (int)(img.getRGB(x, y));
				r = ((rgb >> 16) & 0xFF);
				g = ((rgb >> 8) & 0xFF);
				b = (rgb & 0xFF);
				rgb = (int)((r+g+b)/3);
				//rgb = (int)(0.299 * r + 0.587 * g + 0.114 * b);
				rgb = (255<<24) | (rgb<<16) | (rgb<<8) | rgb;
				grayImage.setRGB(x,y,rgb);
			}
		}
		return grayImage;
	}

	// apply 2x2 pixelation to a grayscale image
	public static BufferedImage pixelate (BufferedImage img) {
		BufferedImage pixImg = new BufferedImage(
			img.getWidth(), img.getHeight(), BufferedImage.TYPE_BYTE_GRAY);
		int pix = 0, p=0;
		for (int y=0; y<img.getHeight()-2; y+=2) {
			for (int x=0; x<img.getWidth()-2; x+=2) {
				pix = (int)((img.getRGB(x, y)& 0xFF)
				+ (img.getRGB(x+1, y)& 0xFF)
				+ (img.getRGB(x, y+1)& 0xFF)
				+ (img.getRGB(x+1, y+1)& 0xFF))/4;
				p = (255<<24) | (pix<<16) | (pix<<8) | pix; 
				pixImg.setRGB(x,y,p);
				pixImg.setRGB(x+1,y,p);
				pixImg.setRGB(x,y+1,p);
				pixImg.setRGB(x+1,y+1,p);
			}
		}
		return pixImg;
	}

	// apply nxn pixelation to a grayscale image
	public static BufferedImage pixelate2 (BufferedImage img, int n) {
		BufferedImage pixImg = new BufferedImage(
			img.getWidth(), img.getHeight(), BufferedImage.TYPE_BYTE_GRAY);
		int pix = 0, p=0;
		for (int y=0; y<img.getHeight()-n; y+=n) {
			for (int x=0; x<img.getWidth()-n; x+=n) {
				for (int a=0; a<n; a++) {
					for (int b=0; b<n; b++) {
						pix += (img.getRGB(x+a, y+b)& 0xFF);
					}
				}
				pix = (int)(pix/n/n);
				for (int a=0; a<n; a++) {
					for (int b=0; b<n; b++) {
						p = (255<<24) | (pix<<16) | (pix<<8) | pix; 
						pixImg.setRGB(x+a,y+b,p);
					}
				}
				pix = 0;
			}
		}
		return pixImg;
	}	

	// scale a grayscale image
	public static BufferedImage resize (BufferedImage img, int newHeight) {
		System.out.println("  Scaling image.");
		double scaleFactor = (double) newHeight/img.getHeight();
		BufferedImage scaledImg = new BufferedImage(
			(int)(scaleFactor*img.getWidth()), newHeight, BufferedImage.TYPE_BYTE_GRAY);
		AffineTransform at = new AffineTransform();
		at.scale(scaleFactor, scaleFactor);
		AffineTransformOp scaleOp = new AffineTransformOp(at, AffineTransformOp.TYPE_BILINEAR);
		return scaleOp.filter(img, scaledImg);
	}

	// apply 3x3 Gaussian blur to a grayscale image
	public static BufferedImage blur (BufferedImage img) {
		BufferedImage blurImg = new BufferedImage(
			img.getWidth()-2, img.getHeight()-2, BufferedImage.TYPE_BYTE_GRAY);
		int pix = 0;
		for (int y=0; y<blurImg.getHeight(); y++) {
			for (int x=0; x<blurImg.getWidth(); x++) {
				pix = (int)(4*(img.getRGB(x+1, y+1)& 0xFF)
				+ 2*(img.getRGB(x+1, y)& 0xFF)
				+ 2*(img.getRGB(x+1, y+2)& 0xFF)
				+ 2*(img.getRGB(x, y+1)& 0xFF)
				+ 2*(img.getRGB(x+2, y+1)& 0xFF)
				+ (img.getRGB(x, y)& 0xFF)
				+ (img.getRGB(x, y+2)& 0xFF)
				+ (img.getRGB(x+2, y)& 0xFF)
				+ (img.getRGB(x+2, y+2)& 0xFF))/16;
				int p = (255<<24) | (pix<<16) | (pix<<8) | pix; 
				blurImg.setRGB(x,y,p);
			}
		}
		return blurImg;
	}

	// apply 5x5 Gaussian blur to a grayscale image
	public static BufferedImage heavyblur (BufferedImage img) {
		BufferedImage blurImg = new BufferedImage(
			img.getWidth()-4, img.getHeight()-4, BufferedImage.TYPE_BYTE_GRAY);
		int pix = 0;
		for (int y=0; y<blurImg.getHeight(); y++) {
			for (int x=0; x<blurImg.getWidth(); x++) {
				pix = (int)(
				10*(img.getRGB(x+3, y+3)& 0xFF)
				+ 6*(img.getRGB(x+2, y+1)& 0xFF)
				+ 6*(img.getRGB(x+1, y+2)& 0xFF)
				+ 6*(img.getRGB(x+2, y+3)& 0xFF)
				+ 6*(img.getRGB(x+3, y+2)& 0xFF)
				+ 4*(img.getRGB(x+1, y+1)& 0xFF)
				+ 4*(img.getRGB(x+1, y+3)& 0xFF)
				+ 4*(img.getRGB(x+3, y+1)& 0xFF)
				+ 4*(img.getRGB(x+3, y+3)& 0xFF)
				+ 2*(img.getRGB(x, y+1)& 0xFF)
				+ 2*(img.getRGB(x, y+2)& 0xFF)
				+ 2*(img.getRGB(x, y+3)& 0xFF)
				+ 2*(img.getRGB(x+4, y+1)& 0xFF)
				+ 2*(img.getRGB(x+4, y+2)& 0xFF)
				+ 2*(img.getRGB(x+4, y+3)& 0xFF)
				+ 2*(img.getRGB(x+1, y)& 0xFF)
				+ 2*(img.getRGB(x+2, y)& 0xFF)
				+ 2*(img.getRGB(x+3, y)& 0xFF)
				+ 2*(img.getRGB(x+1, y+4)& 0xFF)
				+ 2*(img.getRGB(x+2, y+4)& 0xFF)
				+ 2*(img.getRGB(x+3, y+4)& 0xFF)
				+ (img.getRGB(x, y)& 0xFF)
				+ (img.getRGB(x, y+2)& 0xFF)
				+ (img.getRGB(x+2, y)& 0xFF)
				+ (img.getRGB(x+2, y+2)& 0xFF))/74;
				int p = (255<<24) | (pix<<16) | (pix<<8) | pix; 
				blurImg.setRGB(x,y,p);
			}
		}
		return blurImg;
	}

	// detect edges of a grayscale image using Sobel algorithm 
	// (for best results, apply blur before finding edges)
	public static BufferedImage detectEdges (BufferedImage img) {
		int h = img.getHeight(), w = img.getWidth(), threshold=30, p = 0;
		BufferedImage edgeImg = new BufferedImage(w, h, BufferedImage.TYPE_BYTE_GRAY);
		int[][] vert = new int[w][h];
		int[][] horiz = new int[w][h];
		int[][] edgeWeight = new int[w][h];
		for (int y=1; y<h-1; y++) {
			for (int x=1; x<w-1; x++) {
				vert[x][y] = (int)(img.getRGB(x+1, y-1)& 0xFF + 2*(img.getRGB(x+1, y)& 0xFF) + img.getRGB(x+1, y+1)& 0xFF
					- img.getRGB(x-1, y-1)& 0xFF - 2*(img.getRGB(x-1, y)& 0xFF) - img.getRGB(x-1, y+1)& 0xFF);
				horiz[x][y] = (int)(img.getRGB(x-1, y+1)& 0xFF + 2*(img.getRGB(x, y+1)& 0xFF) + img.getRGB(x+1, y+1)& 0xFF
					- img.getRGB(x-1, y-1)& 0xFF - 2*(img.getRGB(x, y-1)& 0xFF) - img.getRGB(x+1, y-1)& 0xFF);
				edgeWeight[x][y] = (int)(Math.sqrt(vert[x][y] * vert[x][y] + horiz[x][y] * horiz[x][y]));
				if (edgeWeight[x][y] > threshold)
					p = (255<<24) | (255<<16) | (255<<8) | 255;
				else 
					p = (255<<24) | (0<<16) | (0<<8) | 0; 
				edgeImg.setRGB(x,y,p);
			}
		}
		return edgeImg;
	}

	// brighten color image by a percentage 
	public static BufferedImage brighten (BufferedImage img, int percentage) {
		int r=0, g=0, b=0, rgb=0, p=0;
		int amount = (int)(percentage * 255 / 100); // rgb scale is 0-255, so 255 is 100%
		BufferedImage newImage = new BufferedImage(
			img.getWidth(), img.getHeight(), BufferedImage.TYPE_INT_ARGB);
		for (int y=0; y<img.getHeight(); y+=1) {
			for (int x=0; x<img.getWidth(); x+=1) {
				rgb = img.getRGB(x, y);
				r = ((rgb >> 16) & 0xFF) + amount;
				g = ((rgb >> 8) & 0xFF) + amount;
				b = (rgb & 0xFF) + amount;
				if (r>255) r=255;
				if (g>255) g=255;
				if (b>255) b=255;
				p = (255<<24) | (r<<16) | (g<<8) | b;
				newImage.setRGB(x,y,p);
			}
		}
		return newImage;
	}
}
```

### outputs

original image
![Alt text](./Image/gaussian_original.png)

gray scale image
![Alt text](./Image/gaussian_grayscale.png)

gaussian filtered (grayscale, kernel 3)
![Alt text](./Image/gaussian_n3.png)

gaussian filtered (grayscale, kernel 7)
![Alt text](./Image/gaussian_n7.png)

gaussian filtered (grayscale, kernel 21)
![Alt text](./Image/gaussian_n21.png)

gaussian filtered (colored, kernel 3)
![Alt text](./Image/gaussian_colored_n3.png)

gaussian filtered (salt and pepper noise, kernel 3)
![Alt text](./Image/gaussian_saltandpepper.png)

***

## Median Filter

### code

code link: [MedianFilter](https://github.com/praserocking/MedianFilter/blob/master/MedianFilter.java)

```java
import java.awt.Color;
import java.awt.image.BufferedImage;
import java.io.*;
import java.util.Arrays;
import javax.imageio.*;
/*
 * Author: Shenbaga Prasanna,IT,SASTRA University;
 * Program: Median Filter To Reduce Noice in Image
 * Date: 9/7/2013
 * Logic: Captures the colour of 8 pixels around the target pixel.Including the target pixel there will be 9 pixels.
 *        Isolate the R,G,B values of each pixels and put them in an array.Sort the arrays.Get the Middle value of the array
 *        Which will be the Median of the color values in those 9 pixels.Set the color to the Target pixel and move on!
 */
public class ImageFilterMedian{
    public static void main(String[] a)throws Throwable{
        String path = "C:\\Users\\solso\\Pictures\\saltandpepper_tiger.png";
        File f=new File(path);                               //Input Photo File
        Color[] pixel=new Color[9];
        int[] R=new int[9];
        int[] B=new int[9];
        int[] G=new int[9];
        File output=new File("MedianOutput.jpg");
        BufferedImage img=ImageIO.read(f);
        for(int i=1;i<img.getWidth()-1;i++)
            for(int j=1;j<img.getHeight()-1;j++)
            {
               pixel[0]=new Color(img.getRGB(i-1,j-1));
               pixel[1]=new Color(img.getRGB(i-1,j));
               pixel[2]=new Color(img.getRGB(i-1,j+1));
               pixel[3]=new Color(img.getRGB(i,j+1));
               pixel[4]=new Color(img.getRGB(i+1,j+1));
               pixel[5]=new Color(img.getRGB(i+1,j));
               pixel[6]=new Color(img.getRGB(i+1,j-1));
               pixel[7]=new Color(img.getRGB(i,j-1));
               pixel[8]=new Color(img.getRGB(i,j));
               for(int k=0;k<9;k++){
                   R[k]=pixel[k].getRed();
                   B[k]=pixel[k].getBlue();
                   G[k]=pixel[k].getGreen();
               }
               Arrays.sort(R);
               Arrays.sort(G);
               Arrays.sort(B);
               img.setRGB(i,j,new Color(R[4],B[4],G[4]).getRGB());
            }
        ImageIO.write(img,"jpg",output);
    }
}
```

### adding salt and pepper noise to image

code link: [salt and pepper](https://github.com/K4m0/Salt-and-Pepper-Noise/blob/master/Main.java)

```java

import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.Random;

import javax.imageio.ImageIO;

import java.awt.Color;
import java.awt.image.BufferedImage;
import java.awt.image.DataBufferByte;
import java.awt.image.Raster;
import java.io.File;
import java.io.IOException;
@SuppressWarnings("unused")
public class Main  {
	
	static BufferedImage image2;
	 static File f = null;
	public static void main(String[] args) {
		try
		{
			//Se carga la imagen
			BufferedImage image1 = ImageIO.read(new File("/Homero.jpg"));
			int width = image1.getWidth();
			int height = image1.getHeight();
			
			//Se convierte a Blanco y Negro
			for(int i=0; i<height; i++){
		         
	            for(int j=0; j<width; j++){
	            
	               Color c = new Color(image1.getRGB(j, i));
	               int red = (int)(c.getRed() * 0.299);
	               int green = (int)(c.getGreen() * 0.587);
	               int blue = (int)(c.getBlue() *0.114);
	               int rgb = range(red+green+blue,8);
	               Color newColor = new Color(rgb,rgb,rgb);
	               
	               image1.setRGB(j,i,newColor.getRGB());
	            }
	         }
			
			try{
			      f = new File("/saved.jpg");
			      ImageIO.write(image1, "jpg", f);
			    }catch(IOException e){
			      System.out.println(e);
			    }

		}
		catch(IOException e)
		{
			System.out.print("No");
		}		
		
	}
	
	public static int range(int n, double prob) {
		double res = ((100 * prob)/10);
		
		int[]array = new int[(int)res];
		array[0]= 1;
		array[1]=255;
		
		for (int i = 2 ; i <= res - 2; i++)
		{
			array[i] = n;
		}
	    int rnd = new Random().nextInt(array.length);
	    return array[rnd];
	}
	
}

	

```

### output

original image
![Alt text](./Image/gaussian_original.png)

salt and pepper noised (color)
![Alt text](./Image/median_colored_input.png)

median filtered (color)
![Alt text](./Image/median_colored_output.png)

salt and pepper noised (gray scale)
![Alt text](./Image/median_grayscale_input.png)

median filtered (gray scale)
![Alt text](./Image/median_grayscale_output.png)
