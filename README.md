# Fractal Autoencoders for Feature Selection


> Feature selection reduces the dimensionality of data by identifying a subset of the most informative features. In this paper, we propose an innovative framework for unsupervised feature selection, called fractal autoencoders (FAE). It trains a neural network to pinpoint informative features for global exploring of representability and for local excavating of diversity. Architecturally, FAE extends autoencoders by adding a one-to-one scoring layer and a small sub-neural network for feature selection in an unsupervised fashion. With such a concise architecture, FAE achieves state-of-the-art performances; extensive experimental results on fourteen datasets, including very high-dimensional data, have demonstrated the superiority of FAE over existing contemporary methods for unsupervised feature selection. In particular, FAE exhibits substantial advantages on gene expression data exploration, reducing measurement cost by about 15% over the widely used L1000 landmark genes. Further, we show that the FAE framework is easily extensible with an application.

The experiments in the paper can be found [here](https://github.com/)

## Model


![](./images/model.png)

Feature selection layer

```sh
class Feature_Select_Layer(Layer):
    
    def __init__(self, output_dim, l1_lambda, **kwargs):
        super(Feature_Select_Layer, self).__init__(**kwargs)
        self.output_dim = output_dim
        self.l1_lambda=l1_lambda

    def build(self, input_shape):
        self.kernel = self.add_weight(name='kernel',  
                                      shape=(input_shape[1],),
                                      initializer=initializers.RandomUniform(minval=0.999999, maxval=0.9999999, seed=seed),
                                      trainable=True,
                                      regularizer=regularizers.l1(self.l1_lambda),
                                      constraint=constraints.NonNeg())
        super(Feature_Select_Layer, self).build(input_shape)
    
    def call(self, x, selection=False,k=key_feture_number):
        kernel=self.kernel        
        if selection:
            kernel_=K.transpose(kernel)
            kth_largest = tf.math.top_k(kernel_, k=k)[0][-1]
            kernel = tf.where(condition=K.less(kernel,kth_largest),x=K.zeros_like(kernel),y=kernel)        
        return K.dot(x, tf.linalg.tensor_diag(kernel))

    def compute_output_shape(self, input_shape):
        return (input_shape[0], self.output_dim)
```


## Usage


```sh
input_img = Input(shape=(p_data_feature,), name='autoencoder_input')

feature_selection = Feature_Select_Layer(output_dim=p_data_feature,\
												l1_lambda=p_l1_lambda,\
												input_shape=(p_data_feature,),\
												name='feature_selection')
												
feature_selection_score=feature_selection(input_img)
feature_selection_choose=feature_selection(input_img,selection=True,k=p_feture_number)
```

For example (see [example](https://github.com/))

![](./images/FAE.png)

![](./images/modelparameters.png)


### Example


#### Testing samples

![](./images/testingdata.png)

#### Loss function and feature selection

<center class="half">
    <img src="./images/loss.gif" width="400"/> &nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img src="./images/allkeyfeatures.gif" width="270"/> </center>
   
#### Key features for MNIST

![](./images/keyfeatures.gif)
   
#### Feconstruction based on selected features

![](./images/rec.gif)

#### key features shown with original samples
 
![](./images/markedkeyfeatures.gif)

   
## Release History

* 0.0.2
    * CHANGE: Update README.md
* 0.0.1
    * ADD: Model and Experiments

## Contacts

Xinxing Wu, Qiang Cheng – Qiang.Cheng@uky.edu

Distributed under the MIT license. See [``LICENSE``](https://github.com/) for more information.