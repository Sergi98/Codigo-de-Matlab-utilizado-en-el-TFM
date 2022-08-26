    clear

    C=imread('calibracion_X.jpg');  % imagen para la calibracion del lote X.
    imshow(C)
    limits=ginput(2); 
    AA(:,1)=(min(limits(:,1)):max(limits(:,1)));
    valory=ceil(mean(limits(:,2)));
    AA(:,2)=valory;
    hold on
    plot(AA(:,1),AA(:,2),'r-')

    Dist=pdist(limits);
    factor_prop=4/Dist;
    Posicion_Feret=1;

    file_path = '.../lotes/LOTE X/';  % ruta de la carpeta de imágenes.
    img_path_list = dir(strcat('.../lotes/LOTE X/','*.jpg'));  % para obtener todas las imágenes en formato jpg de esta carpeta.

    img_num = length (img_path_list);  %para obtener el número total de imágenes.

    if img_num>0  %si se tienen imágenes que cumplen las condiciones.
   
    for j = 1: img_num  % lee imágenes una por una
        image_name = img_path_list(j).name;  % nombre de imagen
        image =  imread(strcat(file_path,image_name));
        fprintf ('% d% d% s \n', j, strcat (file_path, image_name));  % muestra el nombre de la imagen que se está procesando.
        
        I=image;  % se lee la imagen de las partículas.
        I2=rgb2gray(I);  % se pasa la imagen a escala de grises.
        I3=imbinarize(I2,graythresh(I2));  % se binariza la imagen (pasarla a blanco y negro).
        I3=~I3;  % se invierte la imagen.
        I4=imclearborder(I3);  % se eliminan los objetos de los bordes de la imagen.
        A3=bwareaopen(I4,50);  % se filtran los objetos de la imagen (elimina los objetos que tengan menor tamaño del establecido).
        
        L=bwlabel(A3);  % se etiquetan los objetos restantes de la imagen.
        stats=regionprops(L,'Circularity','Area');  % de los objetos ya etiquetados, se obtienen las propiedades deseadas (circualaridad y area).
        
        area=cat(1,stats.Area);
        BW=bwareafilt(A3,[100 15000]);  % filtro del area de las partículas.
        
        L2=bwlabel(BW);
        stats2=regionprops(L2,'Circularity','Area');
        idx=find([stats2.Circularity]>0.8);  % se buscan los objetos que tengan una circularidad mayor o igual a un determinado valor (en este caso 0.8).
        
        BW2=ismember(L2,idx);  % dentro de los objetos etiquetados al principio, se seleccionan los que cumplen la condición de la línea anterior.
        D_Feret=bwferet(BW2);  % se obtienen las propiedades de Feret de las partículas.
        D_Feret=table2array(D_Feret(:,1));
           
        for i=1:length(D_Feret)   % se aplica el proceso para todas las imagenes del lote.
            Feret(Posicion_Feret)=D_Feret(i);  
            Posicion_Feret=Posicion_Feret+1;
        end
        
    end
    
    end

    Feret=factor_prop.*Feret.';  % se multiplica por un factor de proporcionalidad para obtener el tamaño real de las partículas.
 
