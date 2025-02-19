Node Classification
====================

Graph neural networks(GNN) have great power in tackling graph-related tasks. In this chapter, we take node classification
as an example and show how to use CogDL to finish a workflow using GNN. In supervised setting, node classification aims to predict the ground truth
label for each node.

Quick Start
------------
CogDL provides abundant of common benchmark datasets and GNN models. On the one hand, you can simply start a running using
models and datasets in CogDL. This is convenient when you want to test the reproducibility of proposed GNN or get baseline
results in different datasets.

.. code-block:: python

    from cogdl import experiment
    experiment(model="gcn", dataset="cora")



Or you can create each component separately and manually run the process using ``build_dataset``, ``build_model`` in CogDL.

.. code-block:: python

    from cogdl import experiment
    from cogdl.datasets import build_dataset
    from cogdl.models import build_model
    from cogdl.options import get_default_args 

    args = get_default_args(model="gcn", dataset="cora")
    dataset = build_dataset(args)
    model = build_model(args)
    experiment(model=model, dataset=dataset)


As show above, model/dataset/task are 3 key components in establishing a training process. In fact, CogDL also supports
customized model and datasets. This will be introduced in next chapter. In the following we will briefly show the details
of each component.


Save trained model
-------------------

CogDL supports saving the trained model with ``checkpoint_path`` in command line or API usage. For example:

.. code-block:: python

    experiment(model="gcn", dataset="cora", checkpoint_path="gcn_cora.pt")


When the training stops, the model will be saved in `gcn_cora.pt`. If you want to continue the training from previous checkpoint
with different parameters(such as learning rate, weight decay and etc.), keep the same model parameters (such as hidden size, model layers)
and do it as follows:


.. code-block:: python

    experiment(model="gcn", dataset="cora", checkpoint_path="gcn_cora.pt", resume_training=True)


In command line usage, the same results can be achieved with ``--checkpoint-path {path}`` and ``--resume-training``.


Save embeddings
----------------
Graph representation learning (network embedding and unsupervised GNNs) aims to get node representation. The embeddings
can be used in various downstream applications. CogDL will save node embeddings in the given path specified by ``--save-emb-path {path}``. 

.. code-block:: python

    experiment(model="prone", dataset="blogcatalog", save_emb_path="./embeddings/prone_blog.npy")


Evaluation on node classification will run as the end of training. We follow the same experimental settings used in DeepWalk, Node2Vec and ProNE.
We randomly sample different percentages of labeled nodes for training a liblinear classifier and use the remaining for testing
We repeat the training for several times and report the average Micro-F1. By default, CogDL samples 90% labeled nodes for training
for one time. You are expected to change the setting with ``--num-shuffle`` and ``--training-percents`` to your needs.

In addition, CogDL supports evaluating node embeddings without training in different evaluation settings. The following
code snippet evaluates the embedding we get above:

.. code-block:: python

    experiment(
        model="prone",
        dataset="blogcatalog",
        load_emb_path="./embeddings/prone_blog.npy",
        num_shuffle=5,
        training_percents=[0.1, 0.5, 0.9]
    )



You can also use command line to achieve the same results

.. code-block:: bash

    # Get embedding
    python script/train.py --model prone --dataset blogcatalog

    # Evaluate only
    python script/train.py --model prone --dataset blogcatalog --load-emb-path ./embeddings/prone_blog.npy --num-shuffle 5 --training-percents 0.1 0.5 0.9

