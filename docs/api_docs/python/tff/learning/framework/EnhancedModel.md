<div itemscope itemtype="http://developers.google.com/ReferenceObject">
<meta itemprop="name" content="tff.learning.framework.EnhancedModel" />
<meta itemprop="path" content="Stable" />
<meta itemprop="property" content="federated_output_computation"/>
<meta itemprop="property" content="local_variables"/>
<meta itemprop="property" content="non_trainable_variables"/>
<meta itemprop="property" content="trainable_variables"/>
<meta itemprop="property" content="weights"/>
<meta itemprop="property" content="__init__"/>
<meta itemprop="property" content="forward_pass"/>
<meta itemprop="property" content="report_local_outputs"/>
</div>

# tff.learning.framework.EnhancedModel

## Class `EnhancedModel`

Inherits From: [`Model`](../../../tff/learning/Model.md)

A wrapper around a Model that adds sanity checking and metadata helpers.

<h2 id="__init__"><code>__init__</code></h2>

``` python
__init__(model)
```

Initialize self.  See help(type(self)) for accurate signature.



## Properties

<h3 id="federated_output_computation"><code>federated_output_computation</code></h3>

Performs federated aggregation of the a a Model's local_outputs.

This is typically used to aggregate metrics across many clients, e.g. the
body of the computation might be:

    return {
      'num_examples': tff.federated_sum(local_outputs.num_examples),
      'loss': tff.federated_average(local_outputs.loss)}

N.B. It is assumed all TensorFlow computation happens in the
`report_local_outputs` method, and this method only uses TFF
constructs to specify aggregations across clients.

#### Returns:

Either a <a href="../../../tff/Computation.md"><code>tff.Computation</code></a>, or None if no federated aggregation is needed.


The <a href="../../../tff/Computation.md"><code>tff.Computation</code></a> should take as its single input a
<a href="../../../tff.md#CLIENTS"><code>tff.CLIENTS</code></a>-placed <a href="../../../tff/Value.md"><code>tff.Value</code></a> corresponding to the return value of
Model.report_local_outputs, and return a dictionary or other
structure of <a href="../../../tff.md#SERVER"><code>tff.SERVER</code></a>-placed values; consumers of this method should
generally provide these server-placed values as outputs of the overall
computation consuming the model.

<h3 id="local_variables"><code>local_variables</code></h3>

An iterable of `tf.Variable` objects, see class comment for details.

<h3 id="non_trainable_variables"><code>non_trainable_variables</code></h3>

An iterable of `tf.Variable` objects, see class comment for details.

<h3 id="trainable_variables"><code>trainable_variables</code></h3>

An iterable of `tf.Variable` objects, see class comment for details.

<h3 id="weights"><code>weights</code></h3>

Returns a `tff.learning.ModelWeights`.



## Methods

<h3 id="forward_pass"><code>forward_pass</code></h3>

``` python
forward_pass(
    batch,
    training=True
)
```

Runs the forward pass and returns results.

This method should not modify any variables that are part of the model,
that is, variables that influence the predictions; for that, see
`TrainableModel.train_on_batch`.

However, this method may update aggregated metrics computed across calls to
forward_pass; the final values of such metrics can be accessed via
`aggregated_outputs`.

Uses in TFF:

  * To implement model evaluation.
  * To implement federated gradient descent and other
    non-FederatedAvgeraging algorithms, where we want the model to run the
    forward pass and update metrics, but there is no optimizer
    (we might only compute gradients on the returned loss).
  * To implement FederatedAveraging, when augmented as a `TrainableModel`.

TODO(b/120493676): We expect to add another method to this class which
provides access to the shape/dtype/structure expected for the `batch`.

#### Args:

* <b>`batch`</b>: A structure of tensors (as supported by tf.contrib.framework.nest,
    or could be produced by a `tf.data.Dataset`) for the current batch. It
    is the caller's responsibility to provide data of the format expected by
    the Model being called.
* <b>`training`</b>: If True, run the training forward pass, otherwise, run in
    evaluation mode. The semantics are generally the same as the `training`
    argument to `keras.Model.__call__`; this might e.g. influence how
    dropout or batch normalization is handled.


#### Returns:

A BatchOutput namedtuple. This must define a `loss` tensor if the model
will be trained via a gradient-based algorithm.

<h3 id="report_local_outputs"><code>report_local_outputs</code></h3>

``` python
report_local_outputs()
```

Returns tensors representing values aggregated over forward_pass calls.

In federated learning, the values returned by this method will typically
be further aggregated across clients and made available on the server.

This method returns results from aggregating across *all* previous calls
to `forward_pass`, most typically metrics like Accuracy and Loss. If needed,
we may add a `clear_aggregated_outputs` method, which would likely just
run the initializers on the `local_variables`.

In general, the tensors returned can be an arbitrary function of all
the Variables of this model, not just the `local_variables`; for example,
a this could return tensors measuring the total L2 norm of the model
(which might have been updated by training).

This method may return arbitrarily shaped tensors, not just scalar metrics.
For example, it could return the average feature vector or a count of
how many times each feature exceed a certain magnitude.

#### Returns:

A structure of tensors (as supported by tf.contrib.framework.nest)
to be aggregated across clients.


