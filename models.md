# Models

For our evaluation, we implemented three algorithms which are based on three different models (A_DT, A_NN, and A_SVM). In the following, we give details on how we defined those models. Please refer to our paper for more details on how we used those models for blackbox fuzzing.

## Decision Tree

The algorithm A_DT is based on a decision tree (DT) and roughly inspired by the work by Appelt et al. [1]. For this, we directly make use of the `DecisionTreeClassifier` provided by scikit-learn (v1.1.1). We chose the values for the `max_depth` and `max_leaf_nodes` based on the results of a preliminary evaluation with different values for both parameters. In future work, it would be interesting to analyze how much the choice of hyperparameter influences the performance of a fuzzer based on the respective models.

```python
from sklearn.tree import DecisionTreeClassifier

estimator_dt = DecisionTreeClassifier(max_depth = 8, max_leaf_nodes = 40)
```

## Neural Network

The algorithm A_NN is based on a neural network (NN). This idea is based on the work by She et al. [2], and we also base the architecture of the network on their work. As a basis for the implementation, we use Tensorflow (v2.9.1) and Keras. We train the network for 50 epochs.

```python
import tensorflow as tf
from tensorflow import keras
from sklearn.metrics import jaccard_score

MAX_TESTCASE_SIZE = 16
RESULT_SIZE = 5 # in the case of unidimensional feedback, this value is set to 1

estimator_nn = keras.models.Sequential()
estimator_nn.add(keras.layers.Dense(4096, input_dim=MAX_TESTCASE_SIZE))
estimator_nn.add(keras.layers.Activation('relu'))
estimator_nn.add(keras.layers.Dense(RESULT_SIZE))
estimator_nn.add(keras.layers.Activation('sigmoid'))

opt = keras.optimizers.Adam(lr=0.0001)

estimator_nn.compile(loss='binary_crossentropy', optimizer=opt, metrics=[jaccard_score])
```

## Support Vector Machine

The algorithm A_SVM is based on a Support Vector Machine (SVM). We use the implementation provided by scikit-learn v1.1.1 as a basis.

```python
from sklearn import svm
from sklearn.multiclass import OneVsRestClassifier

estimator_svm = OneVsRestClassifier(svm.SVC(probability=True, kernel='linear'))
```

## References

[1] Appelt, D.; Nguyen, C.D.; Panichella, A.; Briand, L.C. A machine-learning-driven evolutionary approach for testing web application firewalls. IEEE Trans. Reliab. 2018, 67, 733–757.

[2] She, D.; Pei, K.; Epstein, D.; Yang, J.; Ray, B.; Jana, S. Neuzz: Efficient fuzzing with neural program smoothing. In Proceedings of the 2019 IEEE Symposium on Security and Privacy (SP), Francisco, CA, USA, 19–23 May 2019; pp. 803–817.