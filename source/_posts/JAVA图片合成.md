-------------------
title: JAVA图片合成
tags: JAVA
categories: Expand
date: 2018-06-27 18:31:49
-------------------

### Description
先说一下我遇到的场景和需求，因为之前一直没有遇到过这种需求，所以在此记录一下，并扩展延伸。   
需求如下：
- 有两张图片，一张大图，一张小图，图片都为链接形式(当然也可以是文件)
- 大图为不同的模板，小图为每个用户特定的二维码(每个用户二维码都不一样)
- 需要将小图缩放到指定大小，然后嵌入在大图的指定位置，大小和位置参数已知。
- 最后合成一张图片上传并返回

### 图片合成方法
直接上代码，在注释中说明

   ```java
    package com.canplay.common.util;
    
    import com.sun.image.codec.jpeg.ImageFormatException;
    
    import javax.imageio.ImageIO;
    import java.awt.*;
    import java.awt.image.BufferedImage;
    import java.io.ByteArrayOutputStream;
    import java.io.IOException;
    import java.net.URL;
    
    /**
     * @author xfan
     * @date Created on 2018/6/27 -- 15:57
     * @desc 图片相关工具类
     */
    public class PictureUtil {
    
        /**
         * 二维码图片合成工具
         * @param x 二维码图片嵌入处X坐标
         * @param y 二维码图片嵌入处Y坐标
         * @param backPicUrl 背景模板图链接
         * @param qrCodePicUrl 二维码链接
         * @param qrCodeHeight 二维码高度
         * @param qrCodeWidth 二维码宽度
         * @return byte 数组
         */
        public static byte[] overlapImage(int x, int y, String backPicUrl, String qrCodePicUrl, int qrCodeHeight, int qrCodeWidth) throws IOException {
            ByteArrayOutputStream out = new ByteArrayOutputStream();
    
            BufferedImage big = ImageIO.read(new URL(backPicUrl));
            //缩放二维码到指定宽高
            BufferedImage small = resize(qrCodeWidth, 
                                         qrCodeHeight, 
                                         ImageIO.read(new URL(qrCodePicUrl)));
    
            Graphics2D g = big.createGraphics();
    
            //将二维码嵌入到坐标位置
            g.drawImage(small, 
                        x,
                        y, 
                        small.getWidth(), 
                        small.getHeight(), 
                        null);
            g.dispose();
            ImageIO.write(big, "jpg", out);
            /*
            * 当然，你也可以直接写出一张图片
            * ImageIO.write(big, "jpg", new File("config/resource/BigSmall.jpg"));
            */
            
            big.flush();
            out.flush();
            out.close();
            
            return out.toByteArray();
        }
    
        /**
         *  将Image的宽度,高度缩放到指定width、height
         * @param width 缩放的宽度
         * @param height 缩放的高度
         * @param targetImage 即将缩放的目标图片
         * @throws ImageFormatException 图片转换异常
         */
        public static BufferedImage resize(int width, int height, Image targetImage) throws ImageFormatException{
            width = Math.max(width, 1);
            height = Math.max(height, 1);
            BufferedImage image = new BufferedImage(width, 
                                                    height, 
                                                    BufferedImage.TYPE_INT_RGB);
            //按照指定大小重新构造图片
            image.getGraphics().drawImage(targetImage, 
                                          0, 
                                          0, 
                                          width, 
                                          height, 
                                          null);
            
            image.flush();
            
            return image;
        }
        
        /**
         * 将Image的宽度、高度缩放到指定width、height，并保存在savePath目录
         * 这个方法就是上面缩放图片方法的另一个版本，写出文件，上面的业务没有用到这个
         * @param width 缩放的宽度
         * @param height 缩放的高度
         * @param savePath 保存目录
         * @param targetImage 即将缩放的目标图片
         * @return 图片保存路径、名称
         * @throws ImageFormatException 图片转换异常
         * @throws IOException IO异常
         */
        public static String resize(int width, int height, String savePath, Image targetImage) throws ImageFormatException, IOException {
            width = Math.max(width, 1);
            height = Math.max(height, 1);
            
            BufferedImage image = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);
            image.getGraphics().drawImage(targetImage, 0, 0, width, height, null);
        		
            if (StringUtils.isBlank(savePath)) {
                //自己定义的默认位置
                savePath = "D:\\new_pic.jpg"; 
            }
            
            //输出流
            FileOutputStream fos = new FileOutputStream(new File(savePath));
            JPEGImageEncoder encoder = JPEGCodec.createJPEGEncoder(fos);
            encoder.encode(image);
         
            image.flush();
            fos.flush();
            fos.close();
        		
            return savePath; 
        }
        
        //下面为延伸...

    }
   ```