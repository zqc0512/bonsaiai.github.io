# Concepts

The `concept` keyword declares an abstract concept that is to be learned by the system. It could be a feature (such as a curvy line in an image) or a goal (such as high score in a game).  Ultimately, a concept takes the form of a transformation of data, but no information need be provided about how to perform the calculation. By declaring a concept, you are instructing the BRAIN server that this is a node in the basic recurrent artificial intelligence network that must be learned. Consequently, concept nodes must have corresponding curricula to teach them.

```inkling--code
concept AbstractConceptName
  is classifier
  predicts ConceptSchema
  follows Antecedent1, Antecedent2   
  feeds Dependent1                 
end
```

Because concepts are learned, their declarations tend to be fairly simple.
Inkling will support the ability to explicitly tell the BRAIN server what learning algorithms and
architecture to use but this is not yet implemented. (It is an unusual case).  

The typical components of a concept statement are shown in the accompanying
panel. 

> Concept Syntax

```inkling--syntax
conceptStatement :=
concept
  is [ classifier | estimator ]
  predicts ( schemaRef )
  [
    follows
      inputSource [',' inputSource ]*
    ]?
  [
    feeds
      outputTarget [',' outputTarget ]*

  ]?
end

inputSource ::=
    input '(' schemaRef? ')' | <name>     # name of a concept or input

outputTarget ::=
    output |  <name>                      # name of a concept or output
```

The `is` keyword specifies the overall class of concept that is being modeled.
For example, a classifier will learn to identify a label corresponding to its
input, an estimator will learn to predict a value, and a predictor (not yet
implemented) will learn sequences and predict subsequent items in a sequence.

The `predicts` keyword declares the concept's output.

The `follows` and `feeds` keywords establish connectivity in the BRAIN directed graph. 

The `follows`, `feeds`, and `predicts` clauses can be in any order. 

Like all Inkling toplevel statements, the `end` keyword declares the end of the
statement. 


### Usage

The concept statement specifies input sources and output targets. Input sources
are listed after the `follows` keyword and output sources are listed after the
`feeds` keyword. 

```inkling--code
concept bar is classifier
  predicts (Move)
  # feeds foo            not necessary 
  follows input(UInt8 x)
end

concept foo is classifier
  predicts (Action)
  follows bar     
  feeds output
end
```

Note that `follows` and `feeds` can be redundant. If our concept foo `follows`
our concept bar then we know that bar is an input source for foo, and it isn't 
necessary to also specify that concept bar `feeds` concept foo.  Specifying foo 
`follows` bar is enough. However, specifying both (foo `follows` bar and bar `feeds` foo) is valid. 

Input sources can be other concepts or the `input` stream.  The `input` stream
is the original input to the system. It flows into the system from outside the
BRAIN. Each reference to the `input` stream must have a schema, which can be an
[anonymous schema][1].

Output targets can also be other concepts or the `output` stream. The `output` stream refers to the output of the BRAIN. 
The `output` stream is never referenced with a schema.

The concept output is described in the `predicts` clause. The schema reference after the
`predicts` keyword describes the data produced by the trained concept.  (This
schema reference is required but it can be [anonymous][1].)
For example, if this concept classifies email into spam and not spam, the output schema for the concept would be a Bool. 

The `is` clause characterizes the output. The `is classifier` form specifies that the output is an enum. 
The `is estimator` form specifies that the output is a value.

Concept input can come from a concept as well as the `input` stream.

When input comes from a concept, the type of the input doesn't matter. This is because concepts don’t act on normal data science data structures. The concept input is a matrix which has no type. 

Concepts do care about their input types when input comes from a stream. This is because the input needs to go through an encoder to become a tensor and the encoder must know the input types. In this case the input types are defined by the output schema of the stream feeding the concept. 

The `predicts` output of a concept is also a matrix which has no type (a
tensor). The `predicts` output schema represents the typed output of the decoder for that output matrix. 

### Examples

We show Inkling for the concepts get_high_score, Digit, Curvature, and Segments. 

**get_high_score**

```inkling--code
concept get_high_score
  is classifier
  predicts PlayerMove
  follows input(GameState)
  feeds output
end
```

* `conceptName`: get_high_score
* `is`: classifier
* `predicts`: PlayerMove
* `input(schemaName)`: input(GameState)
* `feeds`: output


**Digit**

```inkling--code
concept Digit
  is classifier
  predicts MNIST_output
  follows Curvature, Segments, input(MNIST_input)
end
```

* `conceptName`: Digit
* `is`: classifier
* `predicts`: MNIST_output
* `follows`: Curvature and Segments are concepts and `input(MNIST_input)` is
the `input` stream with the MNIST_input schema

**Curvature**

```inkling--code
concept Curvature
  is classifier
  predicts (curve_output)
  follows input(MNIST_input)
end
```

* `conceptName`: Curvature
* `is`: classifier
* `predicts`: curve_output
* `follows`: `input(MNIST_input)`

Note that Digit specifies Curvature in its `follows` clause. That means logically
that Curvature `feeds` Digit. But specifying this relationship in one place is
enough. There would be nothing wrong with adding a `feeds` Digit clause to
Curvature but it isn't required. 

**Segments**

```inkling--code
concept Segments
  is classifier
  predicts (segments_output)
  follows input(MNIST_input)
end
```

* `conceptName`: Segments
* `is`: classifier
* `predicts`: segments_output
* `follows`: `input(MNIST_input)`


[1]: #anonymous-schema
