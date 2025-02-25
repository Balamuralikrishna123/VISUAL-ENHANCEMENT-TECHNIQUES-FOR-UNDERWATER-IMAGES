clc;
clear all;
close all;
clc;
clear all;
close all;
%------------ Image Reading ------------------------------------------
[fname,pname]=uigetfile('*.jpg','Select the Underwater Image');
FPath=strcat(pname,fname);
disp('The Image File Location is');
disp(FPath);
DataArray=imresize(imread(FPath),0.2);
figure,imshow(DataArray);
title('Original image');
% red channel recover
redimg=DataArray(:,:,1);
figure,imshow(redimg);
title('Red channel image');
% red channel compensate
im1 = redCompensate(DataArray,5);
figure,imshow(im1);
title('Red Channel-Image Compensated');
% white balance enhancement
im2 = simple_color_balance(im1);
figure,
imshow(im2);
title('White balance');

52

% gamma correction
input1 = gammaCorrection(im2,1,1.2);
figure,
imshow(input1);
title('gamma correction');
% sharpen
input2 = sharp(im2);
figure,
imshow(input2);
title('Sharpening');
%.................................................%
% calculate weight
%.................................................%
lab1 = rgb_to_lab(input1);
lab2 = rgb_to_lab(input2);
R1 = double(lab1(:, :, 1)/255);
R2 = double(lab2(:, :, 1)/255);
% 1. Laplacian contrast weight (Laplacian filiter on input luminance
channel)
WL1 = abs(imfilter(R1, fspecial('Laplacian'), 'replicate', 'conv'));
WL2 = abs(imfilter(R2, fspecial('Laplacian'), 'replicate', 'conv'));
figure,
imshow(WL2,[]);
title('Laplacian contrast weight')
% 2. Saliency weight
WS1 = saliency_detection(input1);
WS2 = saliency_detection(input2);
figure,
imshow(WS1,[]);

53

title('Saliency weight')
% 3. Saturation weight
WSat1 = Saturation_weight(input1);
WSat2 = Saturation_weight(input2);
figure,
imshow(WS1,[]);
title('Saturation weight')
% normalized weight
[W1, W2] = norm_weight(WL1, WS1, WSat1, WL2 , WS2, WSat2);
figure,
imshow(W1,[]);
title('Normalized weight')
%.................................................%
% image fusion
% R(x,y) = sum G{W} * L{I}
%.................................................%
level = 3;
% weight gaussian pyramid
Weight1 = gaussian_pyramid(W1, level);
Weight2 = gaussian_pyramid(W2, level);
% image laplacian pyramid
% input1
r1 = laplacian_pyramid(double(double(input1(:, :, 1))), level);
g1 = laplacian_pyramid(double(double(input1(:, :, 2))), level);
b1 = laplacian_pyramid(double(double(input1(:, :, 3))), level);
% input2
r2 = laplacian_pyramid(double(double(input2(:, :, 1))), level);
g2 = laplacian_pyramid(double(double(input2(:, :, 2))), level);
b2 = laplacian_pyramid(double(double(input2(:, :, 3))), level);

54

% fusion
for i = 1 : level
R_r{i} = Weight1{i} .* r1{i} + Weight2{i} .* r2{i};
G_g{i} = Weight1{i} .* g1{i} + Weight2{i} .* g2{i};
B_b{i} = Weight1{i} .* b1{i} + Weight2{i} .* b2{i};
end
% pyramin reconstruct
R = pyramid_reconstruct(R_r);
G = pyramid_reconstruct(G_g);
B = pyramid_reconstruct(B_b);
fusion = cat(3, R,G,B);
figure,imshow(fusion,[]),title('Fused image');
% Performance Analysis
uiqm = UIQM(fusion)/10;
disp('UIQM')
disp(uiqm)
uciqe = UCIQE(fusion)/10;
disp('UCIQE')
disp(uciqe)
