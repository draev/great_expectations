---
title: How to create a Custom Column Map Expectation
---
import Prerequisites from '../creating_custom_expectations/components/prerequisites.jsx'

**`ColumnMapExpectations`** are one of the most common types of [**Expectation**](../../../reference/expectations/expectations.md). They are evaluated for a single column and ask a yes/no question for every row in that column. Based on the result, they then calculate the percentage of rows that gave a positive answer. If the percentage is high enough, the Expectation considers that data valid.

This guide will walk you through the process of creating a custom `ColumnMapExpectation`.

<Prerequisites>

- Read the [overview for creating Custom Expectations](./overview.md).

</Prerequisites>

## Steps

### 1. Choose a name for your Expectation

First, decide on a name for your own Expectation. By convention, `ColumnMapExpectations` always start with `expect_column_values_`. You can see other naming conventions in the [Expectations section](/docs/contributing/style_guides/code_style#expectations)  of the code Style Guide.

Your Expectation will have two versions of the same name: a `CamelCaseName` and a `snake_case_name`. For example, this tutorial will use:

- `ExpectColumnValuesToEqualThree`
- `expect_column_values_to_equal_three`

### 2. Copy and rename the template file

By convention, each Expectation is kept in its own python file, named with the snake_case version of the Expectation's name.

You can find the template file for a custom [`ColumnMapExpectation` here](https://github.com/great-expectations/great_expectations/blob/develop/examples/expectations/column_map_expectation_template.py). Download the file, place it in the appropriate directory, and rename it to the appropriate name.

```bash
cp column_map_expectation_template.py /SOME_DIRECTORY/expect_column_values_to_equal_three.py
```

<details>
  <summary>Where should I put my Expectation file?</summary>
  <div>
    <p>
        During development, you don't actually need to put the file anywhere in particular. It's self-contained, and can be executed anywhere as long as <code>great_expectations</code> is installed.
    </p>
    <p>
        But to use your new Expectation alongside the other components of Great Expectations, you'll need to make sure the file is in the right place. The right place depends on what you intend to use it for.
    </p>
    <p>
        <ul>
            <li>If you're building a Custom Expectation for personal use, you'll need to put it in the <code>great_expectations/plugins/expectations</code> folder of your Great Expectations deployment. When you instantiate the corresponding <code>DataContext</code>, it will automatically make all plugins in the directory available for use.</li>
            <li>If you're building a Custom Expectation to contribute to the open source project, you'll need to put it in the repo for the Great Expectations library itself. Most likely, this will be within a package within <code>contrib/</code>: <code>great_expectations/contrib/SOME_PACKAGE/SOME_PACKAGE/expectations/</code>. To use these Expectations, you'll need to install the package.</li>
        </ul>
    </p>
  </div>
</details>

### 3. Generate a diagnostic checklist for your Expectation

Once you've copied and renamed the template file, you can execute it as follows.

```bash
python expect_column_values_to_equal_three.py
```

The template file is set up so that this will run the Expectation's `print_diagnostic_checklist()` method. This will run a diagnostic script on your new Expectation, and return a checklist of steps to get it to full production readiness.

```
Completeness checklist for ExpectColumnValuesToMatchSomeCriteria:
  ✔ Has a library_metadata object
    Has a docstring, including a one-line short description
    Has at least one positive and negative example case, and all test cases pass
    Has core logic and passes tests on at least one Execution Engine
    Has basic input validation and type checking
    Has both Statement Renderers: prescriptive and diagnostic
    Has core logic that passes tests for all applicable Execution Engines
    Passes all linting checks
    Has a robust suite of tests, as determined by a code owner
    Has passed a manual review by a code owner for code standards and style guides
```

When in doubt, the next step to implement is the first one that doesn't have a ✔ next to it. This guide covers the first four steps on the checklist.

### 4. Change the Expectation class name and add a docstring

Let's start by updating your Expectation's name and docstring.

Replace the Expectation class name
```python file=../../../../examples/expectations/column_map_expectation_template.py#L43-L45
```

with your real Expectation class name, in upper camel case:
```python file=../../../../tests/integration/docusaurus/expectations/creating_custom_expectations/expect_column_values_to_equal_three.py#L118
```

You can also go ahead and write a new one-line docstring, replacing
```python file=../../../../examples/expectations/column_map_expectation_template.py#L46
```

with something like:
```python file=../../../../tests/integration/docusaurus/expectations/creating_custom_expectations/expect_column_values_to_equal_three.py#L119
```

You'll also need to change the class name at the bottom of the file, by replacing this line:

```python file=../../../../examples/expectations/column_map_expectation_template.py#L72
```

with this one:
```python file=../../../../tests/integration/docusaurus/expectations/creating_custom_expectations/expect_column_values_to_equal_three.py#L378
```

Later, you can go back and write a more thorough docstring.

At this point you can re-run your diagnostic checklist. You should see something like this:
```
$ python expect_column_values_to_equal_three.py

Completeness checklist for ExpectColumnValuesToEqualThree:
  ✔ Has a library_metadata object
  ✔ Has a docstring, including a one-line short description
    Has at least one positive and negative example case, and all test cases pass
    Has core logic and passes tests on at least one Execution Engine
...
```

Congratulations! You're one step closer to implementing a Custom Expectation.

### 5. Add example cases

Next, we're going to search for `examples = []` in your file, and replace it with at least two test examples. These examples serve a dual purpose:

1. They provide test fixtures that Great Expectations can execute automatically via pytest.

2. They help users understand the logic of your Expectation by providing tidy examples of paired input and output. If you contribute your Expectation to open source, these examples will appear in the Gallery.

Your examples will look something like this:

```python file=../../../../tests/integration/docusaurus/expectations/creating_custom_expectations/expect_column_values_to_equal_three.py#L123-L150
```

Here's a quick overview of how to create test cases to populate `examples`. The overall structure is a list of dictionaries. Each dictionary has two keys:

* `data`: defines the input data of the example as a table/data frame. In this example the table has one column named `all_threes` and a second column named `some_zeroes`. Both columns have 5 rows. (Note: if you define multiple columns, make sure that they have the same number of rows.)
* `tests`: a list of test cases to validate against the data frame defined in the corresponding `data`.
	* `title` should be a descriptive name for the test case. Make sure to have no spaces.
	* `include_in_gallery`: This must be set to `True` if you want this test case to be visible in the Gallery as an example.
	* `in` contains exactly the parameters that you want to pass in to the Expectation. `"in": {"column": "mostly_threes", "mostly": 0.6}` in the example above is equivalent to `expect_column_values_to_equal_three(column=mostly_threes, mostly=0.6)`
	* `out` is based on the Validation Result returned when executing the Expectation.
	* `exact_match_out`: if you set `exact_match_out=False`, then you don’t need to include all the elements of the Validation Result object - only the ones that are important to test.


Run your Expectation file again. The newly added examples won't pass as tests yet, because the Expectation itself hasn't been implemented yet, but they'll check the box for example cases.

```
$ python expect_column_values_to_equal_three.py

Completeness checklist for ExpectColumnValuesToEqualThree:
  ✔ Has a library_metadata object
  ✔ Has a docstring, including a one-line short description
  ✔ Has at least one positive and negative example case, and all test cases pass
    Has core logic and passes tests on at least one Execution Engine
```

:::note
For more information on tests and example cases, <br/>
see our guide on [how to create example cases for a Custom Expectation](../features_custom_expectations/how_to_add_example_cases_for_an_expectation.md).
:::

### 6. Implement your Metric and connect it to your Expectation

This is the stage where you implement the actual business logic for your Expectation.   
To do so, you'll need to implement a function within a [**Metric**](../../../reference/metrics.md) class, and link it to your Expectation.  
By the time your Expectation is complete, your Metric will have functions for all three Execution Engines (Pandas, Spark, & SQLAlchemy) supported by Great Expectations. For now, we're only going to define one.  
  
:::note  
Metrics answer questions about your data posed by your Expectation, <br/> and allow your Expectation to judge whether your data meets ***your*** expectations.  
:::

Your Metric function will have the `@column_condition_partial` decorator, with the appropriate `engine`. Metric functions can be as complex as you like, but they're often very short. For example, here's the definition for a Metric function to calculate whether values equal 3 using the `PandasExecutionEngine`.

```python file=../../../../tests/integration/docusaurus/expectations/creating_custom_expectations/expect_column_values_to_equal_three.py#L49-L51
```

This is all that you need to define for now. The `ColumnMapMetricProvider` and `ColumnMapExpectation` classes have built-in logic to handle all the machinery of data validation, including standard parameters like `mostly`, generation of Validation Results, etc.

<details>
  <summary>Other parameters</summary>
  <div>
    <p>
        <b>Expectation Success Keys</b> - A tuple consisting of values that must / could be provided by the user and defines how the Expectation evaluates success.
    </p>
    <p>
        <b>Expectation Default Kwarg Values</b> (Optional) - Default values for success keys and the defined domain, among other values.
An example of Expectation Parameters is shown below (notice that we are now in a new Expectation class):
    </p>
    <p>
        <b>Metric Condition Value Keys</b> (Optional) - Contains any additional arguments passed as parameters to compute the Metric.
    </p>
  </div>
</details>

Next, choose a Metric Identifier for your Metric. By convention, Metric Identifiers for Column Map Expectations start with `column_values.`. The remainder of the Metric Identifier simply describes what the Metric computes, in snake case. For this example, we'll use `column_values.equal_three`.

You'll need to substitute this metric into two places in the code. First, in the Metric class, replace

```python file=../../../../examples/expectations/column_map_expectation_template.py#L26
```

with

```python file=../../../../tests/integration/docusaurus/expectations/creating_custom_expectations/expect_column_values_to_equal_three.py#L46
```

Second, in the Expectation class, replace

```python file=../../../../examples/expectations/column_map_expectation_template.py#L54
```

with

```python file=../../../../tests/integration/docusaurus/expectations/creating_custom_expectations/expect_column_values_to_equal_three.py#L154
```

It's essential to make sure to use matching Metric Identifier strings across your Metric class and Expectation class. This is how the Expectation knows which Metric to use for its internal logic.

Finally, rename the Metric class name itself, using the camel case version of the Metric Identifier, minus any periods.

For example, replace:

```python file=../../../../examples/expectations/column_map_expectation_template.py#L23
```

with 

```python file=../../../../tests/integration/docusaurus/expectations/creating_custom_expectations/expect_column_values_to_equal_three.py#L43
```

Running your diagnostic checklist at this point should return something like this:
```
$ python expect_column_values_to_equal_three.py

Completeness checklist for ExpectColumnValuesToEqualThree:
  ✔ Has a library_metadata object
  ✔ Has a docstring, including a one-line short description
  ✔ Has at least one positive and negative example case, and all test cases pass
  ✔ Has core logic and passes tests on at least one Execution Engine
...
```

<div style={{"text-align":"center"}}>  
<p style={{"color":"#8784FF","font-size":"1.4em"}}><b>  
Congratulations!<br/>&#127881; You've just built your first Custom Expectation! &#127881;  
</b></p>  
</div>

:::note
If you've already built a [Custom Column Aggregate Expectation](./how_to_create_custom_column_aggregate_expectations.md),
you may notice that we didn't implement a `_validate` method here. While we have to explicitly create this functionality for Column Aggregate Expectations,
Column Map Expectations come with that functionality built in; no extra `_validate` needed!
:::

### 7. Contribution (Optional)

This guide will leave you with a Custom Expectation sufficient for [contribution](../contributing/how_to_contribute_a_custom_expectation_to_great_expectations.md) back to Great Expectations at an Experimental level.

If you plan to contribute your Expectation to the public open source project, you should update the `library_metadata` object before submitting your [Pull Request](https://github.com/great-expectations/great_expectations/pulls). For example:

```python file=../../../../examples/expectations/column_map_expectation_template.py#L63-L69
```

would become

```python file=../../../../tests/integration/docusaurus/expectations/creating_custom_expectations/expect_column_values_to_equal_three.py#L371-L374
```

This is particularly important because ***we*** want to make sure that ***you*** get credit for all your hard work!

:::note
For more information on our code standards and contribution, see our guide on [Levels of Maturity](../../../contributing/contributing_maturity.md#contributing-expectations) for Expectations.

To view the full script used in this page, see it on GitHub:
- [expect_column_values_to_equal_three.py](https://github.com/great-expectations/great_expectations/blob/hackathon-docs/tests/integration/docusaurus/expectations/creating_custom_expectations/expect_column_values_to_equal_three.py)
:::