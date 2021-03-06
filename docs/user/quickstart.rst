==========
Quickstart
==========

Training a model
----------------

Below, we define our own PyTorch ``Module`` and train it on a toy
classification dataset using skorch\'s ``NeuralNetClassifier``:

.. code:: python

    import numpy as np
    from sklearn.datasets import make_classification
    import torch
    from torch import nn
    import torch.nn.functional as F

    from skorch.net import NeuralNetClassifier


    X, y = make_classification(1000, 20, n_informative=10, random_state=0)
    X = X.astype(np.float32)


    class MyModule(nn.Module):
        def __init__(self, num_units=10, nonlin=F.relu):
            super(MyModule, self).__init__()

            self.dense0 = nn.Linear(20, num_units)
            self.nonlin = nonlin
            self.dropout = nn.Dropout(0.5)
            self.dense1 = nn.Linear(num_units, 10)
            self.output = nn.Linear(10, 2)

        def forward(self, X, **kwargs):
            X = self.nonlin(self.dense0(X))
            X = self.dropout(X)
            X = F.relu(self.dense1(X))
            X = F.softmax(self.output(X))
            return X


    net = NeuralNetClassifier(
        MyModule,
        max_epochs=10,
        lr=0.1,
    )

    net.fit(X, y)
    y_proba = net.predict_proba(X)


In an sklearn Pipeline
----------------------

Since ``NeuralNetClassifier`` provides an sklearn-compatible
interface, it is possible to put it into an sklearn ``Pipeline``:

.. code:: python

    from sklearn.pipeline import Pipeline
    from sklearn.preprocessing import StandardScaler


    pipe = Pipeline([
        ('scale', StandardScaler()),
        ('net', net),
    ])

    pipe.fit(X, y)
    y_proba = pipe.predict_proba(X)


Grid search
-----------

Another advantage of skorch is that you can perform an
sklearn ``GridSearchCV`` or ``RandomizedSearchCV``:

.. code:: python

    from sklearn.model_selection import GridSearchCV


    params = {
        'lr': [0.01, 0.02],
        'max_epochs': [10, 20],
        'module__num_units': [10, 20],
    }
    gs = GridSearchCV(net, params, refit=False, cv=3, scoring='accuracy')

    gs.fit(X, y)
    print(gs.best_score_, gs.best_params_)


Notebooks
---------

To see a more elaborate examples, check out the example notebooks
`here
<https://nbviewer.jupyter.org/github/dnouri/skorch/blob/master/notebooks/>`__.
