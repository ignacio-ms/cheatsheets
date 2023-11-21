# Cheatsheet

## Libraries {#lib}

````Python
import cv2
import numpy as np
import matplotlib.pyplot as plt

from skimage.segmentation import slic
from skimage.segmentation import mark_boundaries
````

## Aux functions {#aux-fun}

````Python
negative   = lambda img           : 255 - 1 - img
ampliation = lambda img           : (img - img.min()) * (254 / (img.max() - img.min()))
contrast   = lambda img, k        : img * k
brigthness = lambda img, k        : img + k
gamma_corr = lambda img, k, gamma : k * img ** gamma

def show_image(img, title = None):
    """
    Print given image using OpenCV
    :param img: Image to print
    :param title: Title given to the image
    :return:
    """
    cv2.imshow(title if title is not None else 'Image', img)
    cv2.waitKey(0)
    cv2.destroyAllWindows()
````

## Intensity transformations {#intensity-transformations}

Manual histogram calculation

````Python
def calculaHistograma(img):
    """
    Compute manual histogram
    :param img: Image given
    :return: Histogram of the given image
    """
    h = np.zeros(256)
    f, c = img.shape
    for i in range(f):
        for j in range(c):
            h[img[i,j]] += 1
            
    return h
````

Openvc approach

````Python
imagen = cv2.imread('images/radiografia.jpg', 0)
h = cv2.calcHist([imagen], [0], None, [256], [0,256])
````

Apply the intensity transformation based on the histogram ecualization:

````tex
\sum_{i=0}^{a}h_p(i)
````

Where 
````tex
a=imagen[i,j] and h_p(i)
````

````Python
def ecualization(img, verbose = False):
    """
    Equilize image by appear probability
    :param img: Image to equalize
    :param verbose: Boolean to print extra info
    :return: Equalized image
    """
    img_ec = img.copy()
    h = cv2.calcHist([img], [0], None, [256], [0,256])
    if verbose:
        plt.plot(h)
        plt.show()
    
    f, c = img.shape
    for i in range(f):
        for j in range(c):
            img_ec[i,j] = 255 * np.sum([h[k] for k in range(img[i,j])]) / (f * c)
    
    return img_ec
````

Equalization based on the Probability Density Function (PDF)

````Python
def ecualization_pdf(img, verbose = False):
    """
    Equalize given image by PDF
    :param img: Image to equalize
    :param verbose: Boolean to print extra info
    :return: Equalized image
    """
    h = cv2.calcHist([img], [0], None, [256], [0, 256])
    pdf = np.cumsum(h) / np.size(img)
    if verbose:
        plt.plot(h)
        plt.show()

    img_ec = pdf[img] * 255
    
    return img_ec
````

# Filtering {#filtering}

## Linear filters {#linear-filters}

````Python
def convolution(img, H):
    """
    Apply kernel (Kernel must me normalized, otherwise apply np.mean instead instead of np.sum)
    :param img: Image to aplly convolution
    :param H: Convolution kernel
    :return: Filtered image
    """
    ap_f = H.shape[0] // 2
    ap_c = H.shape[1] // 2
    
    a_image = cv2.copyMakeBorder(img, ap_f, ap_f, ap_c, ap_c, cv2.BORDER_REPLICATE)
    f_image = img.copy()
    
    f, c = img.shape
    for i in range(ap_f, f+ap_f):
        for j in range(ap_c, c+ap_c):
            f_image[i-ap_f, j-ap_c] = np.sum( H * a_image[i-ap_f:i+ap_f+1, j-ap_c:j+ap_c+1] )
            
    return f_image
````

````Python
def im_to_col(img, m, n):
    """
    Transforms an image to a new matrix with a few rows and lot of colomns
    :param img: Image to transform
    :param m: Kernel no. of rows
    :param n: Kernel no. of columns
    :return: Transformed matrix
    """
    ap_f = m//2
    ap_c = n//2
    img = cv2.copyMakeBorder(img, ap_f, ap_f, ap_c, ap_c, cv2.BORDER_REPLICATE)
    
    aux = 0
    f, c = img.shape
    f_img = np.zeros((m*n, f*c))
    for i in range(ap_f, f+ap_f):
        for j in range(ap_c, c+ap_c):
            f_img[:, aux] = img[i-ap_f:i+ap_f+1, j-ap_c:j+ap_c+1].flatten()
            aux += 1
            
    return f_img

def convolution_col(img, H):
    """
    Apply kernel by a column transformation of the original image
    :param img: Image to apply convolution
    :param H: Convolutional kernel
    :return: Filtered image
    """
    f, c = H.shape
    imc = im_to_col(img, f, c)
    
    return np.sum(imc * H.reshape(-1,1), axis = 0).reshape(img.shape)
````

## Non linear filters {#non-linear-filters}

````Python
def non_lineal_filter(img, n, opcion):
    """
    Apply a non lineal filter to a given image
    :param img: Image to apply filter
    :param n: Filter shape (nxn)
    :param opcion: Filter type:
        1: Min_filter
        2: Max_filter
        3: Mean_point_filter
        4: Median filter
    :return: Filtered image
    """
    ap = n // 2
    a_image = cv2.copyMakeBorder(img, ap, ap, ap, ap, cv2.BORDER_REPLICATE)
    f_image = img.copy()
    
    f, c = img.shape
    for i in range(ap, f):
        for j in range(ap, c):
            if opcion == 1:   #Minimun filter
                f_image[i, j] = np.min( a_image[i-ap:i+ap+1, j-ap:j+ap+1] )
                
            elif opcion == 2: #Maximun filter
                f_image[i, j] = np.max( a_image[i-ap:i+ap+1, j-ap:j+ap+1] )
                        
            elif opcion == 3: #Mean point filter
                f_image[i, j] = np.mean((np.min(a_image[i-ap:i+ap+1, j-ap:j+ap+1]), np.max(a_image[i-ap:i+ap+1, j-ap:j+ap+1])))
                
            else:             #Median filter
                f_image[i, j] = np.median(a_image[i-ap:i+ap+1, j-ap:j+ap+1].ravel())
    
    return f_image
````

## Non Homogeneous filter {#non-homogeneous-filter}

````Python
def non_homogeneous_filter(img, n):
    """
    Apply a non homogeneous to a given image
    :param img: Image to apply filter
    :param n: Filter shape (nxn)
    :return: Filtered image
    """
    ap = n // 2
    a_image = cv2.copyMakeBorder(img, ap, ap, ap, ap, cv2.BORDER_REPLICATE)
    f_image = img.copy()
    
    f, c = img.shape
    for i in range(ap, f-ap):
        for j in range(ap, c-ap):
            H = np.zeros(shape = (n, n))
            H[np.random.randint(n), np.random.randint(n)] = 1
            f_image[i, j] = np.sum(H * a_image[i-ap:i+ap+1, j-ap:j+ap+1])
 
    return f_image
````

````Python
def adaptative(img, Smax):
    """
    Aplly an adaptative filter to a given image. The filter eliminates extreme pixel intensities
    increasing the kernel shape by each iteration.
    :param img: Image to apply kernel
    :param Smax: Maximun shape of the increasing kernel (Smax x Smax)
    :return: Filtered image
    """
    ap_max = Smax // 2
    a_imagen = cv2.copyMakeBorder(img, ap_max, ap_max, ap_max, ap_max, cv2.BORDER_REPLICATE)
    f_imagen = img.copy()
    
    f, c = img.shape
    for i in range(f):
        for j in range(c):
            
                ap_aux = 1
                e = img[i, j]
                while ap_aux <= ap_max and (e == 0 or e == 255):
                    
                    e = np.median(a_imagen[i-ap_aux:i+ap_aux+1, j-1:j+ap_aux+1])
                    ap_aux += 1
                    
                f_imagen[i, j] = e
    
    return f_imagen
````

# Canny edge detector {#canny}

Filter to get rid of noise

````Python
def convolution(img, H):
    """
    Apply kernel (Kernel must me normalized, otherwise apply np.mean instead instead of np.sum)
    :param img: Image to aplly convolution
    :param H: Convolution kernel
    :return: Filtered image
    """
    ap_f = H.shape[0] // 2
    ap_c = H.shape[1] // 2
    
    a_image = cv2.copyMakeBorder(img, ap_f, ap_f, ap_c, ap_c, cv2.BORDER_REPLICATE)
    f_image = img.copy()
    
    f, c = img.shape
    for i in range(ap_f, f+ap_f):
        for j in range(ap_c, c+ap_c):
            f_image[i-ap_f, j-ap_c] = np.sum( H * a_image[i-ap_f:i+ap_f+1, j-ap_c:j+ap_c+1] )
            
    return f_image
````

Border detection by gradient calculation.

Convolution masks:

m_g_x = [[-1, -2, -1], [0, 0, 0], [1, 2, 1]]

m_g_y = [[-1, 0, 1], [-2, 0, 2], [-1, 0, 1]]

Magnitude image applying:

````tex
E(u,v)=\sqrt{I_x (u,v)^2+I_y (u,v)^2 }
````

And orientation image applying:

````tex
\Phi(u,v)=atan2(I_y (u,v),I_x (u,v))
````

````Python
def gradient(img):
    """
    Calculates the Magnitude and Orientation of the gradient of a given image
    :param img: Image to calculate gradient
    :return: Magnitude and Orientation matrix
    """
    m_g_x = np.array([[-1, 0, 1], [-2, 0, 2], [-1, 0, 1]])
    m_g_y = np.array([[1, 2, 1], [0, 0, 0], [-1, -2, -1]])
    
    img_x = convolution(img, m_g_x)
    img_y = convolution(img, m_g_y)
    
    E   = np.hypot(img_x, img_y)
    Phi = np.rad2deg(np.arctan2(img_y, img_x))
    
    return E, Phi
````

Non maximums suppression

````Python
def non_max_supression(M, alpha):
    """
    Supression of non max pixels depending on the gradient of the pixel's neighbours
    :param M: Gradient Magnitude matrix
    :param alpha: Gradient Orientation matrix
    :return: Supressed image
    """
    EN = M.copy()
    alpha[alpha < 0] += 180
    
    f, c = M.shape
    for i in range(1, f-1):
        for j in range(1, c-1):
            
            q, r = 255, 255
            
            # 0º
            if (0 <= alpha[i, j] < 22.5) or (157.5 <= alpha[i, j] <= 180):
                q = M[i, j+1]
                r = M[i, j-1]
                
            # 45º
            elif 22.5 <= alpha[i, j] < 67.5:
                q = M[i+1, j-1]
                r = M[i-1, j+1]
                
            # 90º
            elif 67.5 <= alpha[i, j] < 112.5:
                q = M[i+1, j]
                r = M[i-1, j]
                
            # 135º
            elif 112.5 <= alpha[i, j] < 157.5:
                q = M[i-1, j-1]
                r = M[i+1, j+1]
                
            # Neighbours
            if M[i, j] == np.max([M[i, j], q, r]):
                EN[i, j] = M[i, j]
            else:
                EN[i, j] = 0
    
    return EN
````

Thresholding and gaps filling

Choose thresholds like:

````tex
q_l=0.1  \max_{(u,v)} E_N (u,v) 
````

````tex
q_h=0.3   \max_{(u,v)} E_N (u,v) 
````

Build images as:

````tex
E_{NL} (u,v)=E_N (u,v)\ge q_l \mbox{ and }E_N(u,v)< q_h
````
and 
````tex
E_{NH} (u,v)=E_N (umv)\ge q_h
````


````Python
def thresholding_and_filling(EN):
    """
    Thresholding the image "eliminating" the pixels below a gigen minimun threshold, "keeping" tho ones over a
    maximun threshold and Filling by keeping remaining ones if one of his neighbours were already keeped
    :param EN: Image with no max pixels
    :return: Edge image
    """
    # Thresholding
    ql, qh = 0.1 * np.max(EN), 0.2 * np.max(EN)
    f, c = EN.shape
    contornos = np.zeros((f, c))
    
    w, s = 25, 255
    
    s_i, s_j = np.where(EN >= qh)
    w_i, w_j = np.where((EN >= ql) & (EN <= qh))
    
    contornos[s_i, s_j] = s
    contornos[w_i, w_j] = w
    
    # Filling
    for i in range(1, f-1):
        for j in range(1, c-1):
            if contornos[i, j] == w:
                if np.max(contornos[i-1: i+2, j-1:j+2]) == s:
                    contornos[i, j] = s
                else:
                    contornos[i, j] = 0
    
    return contornos


def canny_detector(image, n):
    """
    Implementation of canny detector
    :param image: Image to filter
    :param n: Shape of the gaussian kernel (nxn)
    :return: Edge image
    """
    sigma = 1.4 #desviación de la gaussiana
    mask = cv2.getGaussianKernel(n, sigma)*cv2.getGaussianKernel(n, sigma).T

    f_image = convolution(image, mask)
    E, Phi = gradient(f_image)
    EN = non_max_supression(E, Phi)
    edge_image = thresholding_and_filling(EN)
    
    return edge_image
````

# Thresholding {#thresholding}

## Min Between Max {#min-b-max}

````Python
def min_b_max(img):
    """
    Threshold a given image using as threshold the minimun intensity between the 2 intensity peaks of the histogram
    :param img: Image to threshold
    :return: Thresholded image
    """
    h = cv2.calcHist([img], [0], None, [256], [0, 256])

    q1 = np.argmax(h)
    q2 = 0
    for i in range(h.size):
        aux = h[i] * np.abs( i - q1 )
        if aux > h[q2]:
            q2 = i

    q = np.argmin(h[q1:q2] if q1<q2 else h[q2:q1]) + (q1 if q1<q2 else q2)
    t_img = np.array(255 * (img <= q)).astype(np.uint8)

    return t_img
````

## Otsu {#otsu}

````Python
def otsu(img):
    """
    Otsu thresholding function implementation. 
    Better performance applying gaussian kernel
    :param img: Image to threshold
    :return: Thresholded image
    """
    h = cv2.calcHist([img], [0], None, [256], [0, 256])
    sigma_arr = np.zeros(h.size)

    for i in range(1, h.size-1):
        p0, p1 = np.sum(h[:i]) / img.size , np.sum(h[i:]) / img.size

        mu0 = np.sum([j * h[j] for j in range(i)]) / np.sum(h[:i])
        mu1 = np.sum([j * h[j] for j in range(i, h.size)]) / np.sum(h[i:])

        sigma_arr[i] = p0 * p1 * (mu0 - mu1) ** 2

    q = np.argmax(sigma_arr)
    t_img = np.array(255 * (img <= q)).astype(np.uint8)

    return t_img
````

## Minimun error Thresholding {#min-err}

````Python
gaus = lambda x, mu, sigma : (1 / np.sqrt(2*mu*sigma)) * np.exp(-(((x - mu)**2) / (2*sigma)))

def minimun_error(img, verbose = False):
    """
    Minimum error thresholding function implementation
    :param img: Image to threshold
    :param verbose: Boolean for printing extra info
    :return: Thresholded image
    """
    h = cv2.calcHist([img], [0], None, [256], [0, 256])
    err = np.zeros(h.size-2)

    for i in range(1, h.size-1):
        p0, p1   = np.sum(h[:i]) / img.size , np.sum(h[i:]) / img.size

        mu0 = np.sum([j * h[j] for j in range(i)]) / np.sum(h[:i])
        mu1 = np.sum([j * h[j] for j in range(i, h.size)]) / np.sum(h[i:])

        sigma0 = np.sum([(j - mu0)**2 * h[j] for j in range(i)]) / np.sum(h[:i]) + 1e-10
        sigma1 = np.sum([(j - mu1)**2 * h[j] for j in range(i, 256)]) / np.sum(h[i:]) + 1e-10

        e0 = np.sum([((j - mu0)** 2 / sigma0) + 2*(np.log(sigma0) - np.log(p0 + 1e-10)) for j in range(i)])
        e1 = np.sum([((j - mu1)** 2 / sigma1) + 2*(np.log(sigma1) - np.log(p1 + 1e-10)) for j in range(i+1, 256)])

        err[i-1] = p0 * e0 + p1 * e1

    q = np.argmin(err)
    if verbose:
        print('Threshold: ', q)

        mu0 = np.sum([j * h[j] for j in range(q)]) / np.sum(h[:q])
        mu1 = np.sum([j * h[j] for j in range(q, h.size)]) / np.sum(h[q:])

        sigma0 = np.sum([(j - mu0)**2 * h[j] for j in range(q)]) / np.sum(h[:q]) + 1e-10
        sigma1 = np.sum([(j - mu1)**2 * h[j] for j in range(q, 256)]) / np.sum(h[q:]) + 1e-10

        plt.plot(h)
        plt.show()
        x0 = np.arange(q)
        plt.plot(x0, gaus(x0, mu0, sigma0))

        x1 = np.arange(q, 256)
        plt.plot(x1, gaus(x1, mu1, sigma1))
        plt.show()

    t_img = np.array(255 * (img<= q)).astype(np.uint8)
    return t_img
````

## Variable thresholding {#var-t}

````Python
def variable(img):
    """
    Variable thresholding function implementation
    :param img: Image to threshold
    :return: Thresholded image
    """
    a_img = cv2.copyMakeBorder(img, 1, 1, 1, 1, cv2.BORDER_REPLICATE)
    f_img = img.copy()
    
    f, c = img.shape
    for i in range(1, f+1):
        for j in range(1, c+1):
            mu = np.mean( a_img[i-1:i+1, j-1:j+2] )
            sigma = np.std( a_img[i-1:i+1, j-1:j+2] )
            
            f_img[i-1, j-1] = 1 if img[i-1, j-1] < (sigma + (mu / 2)) else 0
    
    return f_img
````

# Clustering {#clustering}

## K-means implementation (Standarized) {#k-means}

````Python
def mean_filter(img, S):
    """
    Apply a mean filter of a given image 
    :param img: Image to filter
    :param S: Kernel's shape (SxS)
    :return: Mean filtered image
    """
    a = S//2

    a_img = cv2.copyMakeBorder(img, a, a, a, a, cv2.BORDER_REPLICATE)
    f_img = np.zeros(img.shape)

    f, c = img.shape[0:2]
    for i in range(a, f+a):
        for j in range(a, c+a):
            f_img[i-a, j-a] = a_img[i-a:i+a+1, j-a: j+a+1].mean(axis = (0, 1))

    return f_img


def random_centers(img, K):
    """
    Calculates K random centers of clusters
    :param img: Image
    :param K: Numer of centers
    :return: Mean filtered image
    """
    f, c = img.shape[0:2]
    index_x = np.random.randint(low=0, high=f, size=K)
    index_y = np.random.randint(low=0, high=c, size=K)

    return img[index_x, index_y]


def classify(img, mean_img, K, centers, alpha):
    """
    Calculates a label image indicating for each pixel at witch cluster it belongs
    :param img: Image being processed
    :param mean_img: Mean image
    :param K: No. of clusters
    :param centers: Cluster centers
    :param alpha: Standardization constant
    :return: Label image
    """
    dist_aux = np.empty((img.shape[0], img.shape[1], K))

    for i in range(K):
        dist_aux[:,:,i] = np.sum(np.abs(img - centers[i])**2, axis=2) \
                          + alpha*np.sum(np.abs(mean_img - centers[i])**2, axis=2)

    return dist_aux.argmin(axis=2)


def cost(img, mean_img, labels, centers, alpha):
    """
    Compute cost function
    :param img: Image being processed
    :param mean_img: Mean image
    :param labels: Labels of cluster typyes
    :param centers: Cluster centers
    :param alpha: Standardization constant
    :return: Cost
    """
    op = np.sum(np.sum(np.abs(img - centers[labels])**2, axis=2))
    return op + alpha*np.sum(np.sum(np.abs(mean_img - centers[labels])**2, axis=2))


def update_centers(img, mean_img, labels, centers, K, alpha):
    """
    Update Cluster centers minimizing cost
    :param img: Image being processed
    :param mean_img: Mean image
    :param labels: Labels of cluster types
    :param centers: Cluster centers
    :param K: Np. of clusters
    :param alpha: Standardization constant
    :return: New centers
    """
    for i in range(K):
        centers[i] =  np.sum(img[labels==i, :] + alpha*mean_img[labels==i, :], axis=0) / (len(img[labels==i]) * (1+alpha))

    return centers


def final_image(img, labels, K):
    """
    Create a new image assigning each pixel the mean color of its cluster
    :param img: Image to filter
    :param labels: Label image of cluster classes
    :param K: No. of clusters
    :return: Filtered image
    """
    f_img = img.copy()

    for i in range(K):
        f_img[labels==i, :] = np.mean(img[labels==i, :], axis=0)

    return f_img


def kmeans_standarized(img, K, alpha, S):
    """
    Implementation of k-means algorithm using Standardization
    :param img: Image to process
    :param K: No. of clusters
    :param alpha: Standardization constant
    :param S: Mean filter kernel shape (SxS)
    :return: Label matrix, Cluster centers, Segmented image
    """
    img = np.float32(img)

    medias = mean_filter(img, S)
    centros = random_centers(img, K)
    labels = classify(img, medias, K, centros, alpha)

    coste_ant = np.Infinity
    coste_act = cost(img, medias, labels, centros, alpha)

    while (coste_ant - coste_act) > 10**-5:
        coste_ant = coste_act

        centros = update_centers(img, medias, labels, centros, K, alpha)
        labels = classify(img, medias, K, centros, alpha)
        coste_act = cost(img, medias, labels, centros, alpha)

        # print(coste_act)

    segmentada = final_image(img, labels, K)
    return labels, centros, segmentada
````

## Superpixels {#superpixels}

````Python
img = cv2.imread('images/playa.jpg')

K = 1200
super_p = slic(img.copy(), n_segments=K, start_label=0)

f_img_1 = mark_boundaries(img.copy(), super_p)
f_img_2 = final_image(img.copy(), super_p, K)

cv2.imshow('Superpixels_boundaries', f_img_1)
cv2.imshow('Superpixels_mean_colors', f_img_2)
cv2.waitKey(0)
cv2.destroyAllWindows()
````
