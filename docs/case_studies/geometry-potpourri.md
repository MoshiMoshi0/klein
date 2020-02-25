!!! danger "ARTICLE CURRENTLY BEING DRAFTED"

In this workshop, we're going to walk through a number of common geometric tasks
such as finding intersections, computing distances, performing rotations, etc. For each task, we'll
showcase not only the Klein code that would produce the desired result, but also show mathematically
how it can be computed by hand.

!!! danger "Lack of rigor ahead!"

    In this article, I make very little effort to explain why an operation is defined a certain way or explain some of the deeper underlying mathematics. This article was written because it's often helpful to see how computation is done first, and perform some calculations by hande as well gain some intuition before taking a closer look at the theory.

## Multivectors

!!! info

    Klein API: [kln::entity](../../api/kln_entity)

In this section, we look at all the various operations and elements that show up in $\PGA$.
A common question that people run into when getting exposed to GA for the first time is, "why are
their so many operations?" The best answer is that, well, geometry has a lot of operations! More
than can be captured easily with just the dot product and cross product you're likely familiar with.
This section just gets you acquainted with the mechanics of $\PGA$, but to glean more geometric
intuition, _all_ the sections afteward will focus on concrete examples (plane distance to point,
meeting to points to make a line, projecting a line onto a plane, rotating a point about a line,
reflecting a point through a plane, etc). You'll find that for a bit of upfront complexity presented
here, what's gained is a satisfying degree of uniformity in all the geometry that comes later.

### PGA Basis and Geometric Product

!!! info

    Klein API: `operator*`

    Written as two elements adjacent to each other ($ab$).

Let's get used to some basis elements and operations first.
In $\mathbf{P}(\mathbb{R}^*_{3, 0, 1})$ (also known as PGA), we have four grade-1 vector basis
elements, $\ee_0$, $\ee_1$, $\ee_2$, and $\ee_3$.
Let's define the square of these vectors in the following manner.
The first element squares to $0$ ($\ee_0 \ee_0 =
0$) while the latter 3 elements square to $1$ ($\ee_1\ee_1 = \ee_2\ee_2 = \ee_3\ee_3 = 1$).
As we now immediately have a means to create $1$, we can generate all the numbers this way,
so let's declare $1$ as our grade-0 basis element.
When we write elements adjacent to one another as in $\ee_1\ee_1$, the operation being represented
here is the _geometric product_. In code, the geometric product is expressed as the multiplication
(`*`) operator. So far, we've written down what the geometric product does when the
elements multiply against themselves. Next, we need to describe what happens when we take the
geometric product between different elements.

Mixed products of the basis vectors produce the basis bivectors, which can themselves be used in
linear combinations to produce general bivectors. Bivectors that can be written as the product of
two vectors may be referred to as _blades_ or _simple bivectors_. The products between basis vectors
are all blades by definition. As shorthand, we combine subscripts as shown
below:

$$
\begin{aligned}
\ee_1\ee_0 &= \ee_{10} \\
\ee_1\ee_2 &= \ee_{12} \\
\ee_3\ee_0 &= \ee_{30} \\
&\dots
\end{aligned}
$$

At the moment, we would have $4\times 3 = 12$ basis bivectors if defined this way, but we impose a
relation on them that relates basis bivectors that contain the same subscripts. Namely, the product
of two vectors is minus the product of the two vectors with the order interchanged.

$$
\begin{aligned}
\ee_{01} &= -\ee_{10} \\
\ee_{02} &= -\ee_{20} \\
\ee_{03} &= -\ee_{30} \\
\ee_{12} &= -\ee_{21} \\
\ee_{31} &= -\ee_{13} \\
\ee_{23} &= -\ee_{32} \tag{1}
\end{aligned}
$$

For reasons that will become clear in a future writeup, we'll select the blades on the left hand side of $(1)$ to be the
_canonical 2-blades_ in PGA. There is some choice index order selection as will be explained later,
but the order here is used to match existing literature and so that computations later are made more
convenient.

Now that we know we can swap the order of the multiplication by basis vectors by introducing a sign,
we can compute things like $\ee_{01}\ee_0$.

$$
\begin{aligned}
\ee_{01}\ee_0 &= \ee_0\ee_1\ee_0 \\
&= -\ee_0\ee_0\ee_1 \\
&= 0
\end{aligned}
$$

Another example (that doesn't vanish):

$$
\begin{aligned}
\ee_{20}\ee_2 &= \ee_2\ee_0\ee_2 \\
&= -\ee_2\ee_2\ee_0 \\
&= -\ee_0
\end{aligned}
$$

The same algorithm can be used to produce the 4 trivectors:

$$
\begin{aligned}
\ee_{123} &= \ee_{12}\ee_3 = -\ee_{132} = \ee_{312} \\
\ee_{021} &= \ee_{02}\ee_1 = -\ee_{201} = \ee_{210} \\
\ee_{013} &= \ee_{01}\ee_3 = -\ee_{031} = \ee_{301} \\
\ee_{032} &= \ee_{03}\ee_2 = -\ee_{302} = \ee_{320}
\end{aligned}
$$

Not all permutations were listed, but just a few so you can gain familiarity with swapping indices
and introducing signs or removing them as ncessary. The last element we haven't touched on is the
pseudoscalar $\ee_{0123}$. The pseudoscalar is so named because while it does not have grade zero,
it is uniquely identified within its grade to within a permutation of its indices (compared to, say,
the grade-2 elements of which there are six distinct members).
Note that there is no way in which we can introduce an element with grade
higher than this, and so our $16$ algebra elements (1 scalar, 4 vector, 6 bivector, 4 trivector, and
1 pseudoscalar) are fully described. Linear combinations can be made from all of these elements to
produce a general multivector. To take the geometric product between general multivectors, we
leverage the fact that the geometric product obeys the distributive law.

$$
\begin{aligned}
(4\ee_1 - 2\ee_{32})(3\ee_{012} + \ee_{10}) &= 4\ee_1(3\ee_{012} + \ee_{10}) - 2\ee_{32}(3\ee_{012}
+ \ee_{10}) \\
&= 12\ee_1\ee_{012} + 4\ee_1\ee_{10} - 6\ee_{32}\ee_{012} - 2\ee_{32}\ee_{10} \\
&= -12\ee_{02} + 4\ee_0 - 6\ee_{301} - 2\ee_{3210} \\
&= 4\ee_0 - 12\ee_{02} - 6\ee_{013} - 2\ee_{0123}
\end{aligned}
$$

In the last step, we simply took the time to sort the resulting terms by grade and also performed
swaps as necessary to write terms with the canonical subscript ordering.

!!! tip "Exercises"

    1. Write down two vectors whose product is $-\ee_{01} + 2\ee_{13}$.
    2. The geometric product is neither commutative, nor anticommutative. Find examples that demonstrate this fact.
    3. Not every bivector can be written down as the product of two vectors. Find an example that demonstrates this fact.

### The Symmetric Inner Product

!!! info

    Klein API: `operator|`

    Written as $a \cdot b$.

The _symmetric inner product_ (inner product for short) is similar to the geometric product except
it _is always grade decreasing_ such that the final grade is the absolute value of the difference of
the operand grades. If the grade of the element resulting from a geometric product
would have been greater than this difference, the inner product extinguishes it to zero. In other
words, all indices must contract for the inner product to produce a non-zero value (note that if the
degenerate element contracts, a zero is produced anyways).
Before, we
saw that the geometric product can decrease the grade in
the event that indices match (which contracts the result to $1$ or $0$ depending). The easy way to
perform a symmetric product is to perform the geometric product as normal but discard terms produced
that produce the undesired grade. For example, $\ee_1\cdot \ee_1 = 1$ but $\ee_1\cdot
\ee_2 = 0$.
For a more nuanced example, $\ee_{12}\cdot\ee_{02}$ is _also_ $0$ because the $1$ index did not contract.
Note that we write the symmetric inner product as $\cdot$ in equation form, but in code,
it is expressed via the pipe (`|`) operator. As with the geometric product, the symmetric inner
product obeys the distributive law. Here's an example mirroring an example above but using the inner
product instead of the geometric product:

$$
\begin{aligned}
(4\ee_1 - 2\ee_{32})\cdot (3\ee_{012} + \ee_{10}) &= 4\ee_1\cdot(3\ee_{012} + \ee_{10}) -
2\ee_{32}\cdot (3\ee_{012} + \ee_{10}) \\
&= -12 \ee_{02} + 4\ee_0 \\
&= 4\ee_0 - 12\ee_{02}
\end{aligned}
$$

In addition, because of our "rule" that the inner product can only decrease grade, the inner product
between any quantity and a scalar quantity must be zero.

!!! tip "Exercises"

    1. Can the symmetric inner product ever produce the pseudoscalar? Why or why not?
    2. Explain why $\ee_0 \cdot \ee_{012}$ is $0$ and not $\ee_{12}$.
    3. The symmetric inner product not always commutative! Provide a counterexample for the inner product between other grades. When is it commutative?

### The Exterior Product

!!! info

    Klein API: `operator^`

    Written as $a\wedge b$.

The _exterior product_ (also known as the _wedge_ product or the _outer_ product) is similar to the
geometric product except it always extinguishes results upon contraction.
Thus, the result of an exterior product either has grade equal to the sum of the grades of its operands, or it
is exactly zero. The exterior product is represented in equations with a $\wedge$ symbol, and in
code, is represented with a caret (`^`) operator. The exterior product obeys the laws of distributivity as with the other products,
so let's return to our familiar example:

$$
\begin{aligned}
(4\ee_1 - 2\ee_{32})\wedge (3\ee_{012} + \ee_{10}) &= 4\ee_1\wedge(3\ee_{012} + \ee_{10}) -
2\ee_{32}\wedge (3\ee_{012} + \ee_{10}) \\
&= -2\ee_{3210} \\
&= -2\ee_{0123}
\end{aligned}
$$

!!! tip "Exercises"

    1. The geometric product **between vectors** (grade-1) is the sum of the inner product and the exterior product.
       Prove this by explicitly expanding out the geometric product between two general vectors.
    2. The geometric product is **not** generally the sum of the inner product and the exterior product. Why is that?
    3. The exterior product isn't generally anti-commutative! Find counterexamples demonstrating this fact.

### The Poincaré Dual Map

!!! info

    Klein API: `operator!`

    Written as $\JJ(a)$.

To perform the regressive product in the next section, we need a map that is _grade-reversing_. That
is, we need a map that takes multivectors $v$ to multivectors $v^*$ such that the grade of $v$ is
equal to $4$ minus the grade of $v^*$ (and vice versa). For example, vectors should be mapped to
trivectors, bivectors to bivectors, and the scalar to the pseudoscalar. In addition, this map needs
to be an involution (two application of the map is the identity). Here, a map is provided but note
that several such maps are possible. The nice property of the map provided here is that the
application of the map is just a coordinate tuple reversal without any sign changes. The mechanism for how this basis
was chosen is elegant but will be the subject of a different article. By convention, the dual
map is written as $\JJ$ in expressions.

$$
\begin{aligned}
    \JJ(1) = \ee_{0123}&,\quad \JJ(\ee_{0123}) = 1 \\
    \JJ(\ee_0) = \ee_{123}&,\quad \JJ(\ee_{123}) = \ee_0 \\
    \JJ(\ee_1) = \ee_{032}&,\quad \JJ(\ee_{032}) = \ee_1 \\
    \JJ(\ee_2) = \ee_{013}&,\quad \JJ(\ee_{013}) = \ee_2 \\
    \JJ(\ee_3) = \ee_{021}&,\quad \JJ(\ee_{021}) = \ee_3 \\
    \JJ(\ee_{01}) = \ee_{23}&,\quad \JJ(\ee_{23}) = \ee_{01} \\
    \JJ(\ee_{02}) = \ee_{31}&,\quad \JJ(\ee_{31}) = \ee_{02} \\
    \JJ(\ee_{03}) = \ee_{12}&,\quad \JJ(\ee_{12}) = \ee_{03}
\end{aligned}
$$

!!! tip "Exercises"

    1. Double check that all elements in the algebra are accounted for above.
    2. The dual map is a unary operator that can be distributed across each term in a multivector sum. For example, $\JJ(\ee_1 + 2\ee_{02}) = \ee_{032} + 2\ee_{31}$. Verify that the map is still an involution (that is, two applications of $\JJ$ should map back to the original argument).

### Regressive Product

!!! info

    Klein API: `operator&`

    Written as $a\vee b$.

The _regressive product_ (written as $\vee$ and expressed as the `&` operator in code) is defined in
terms of the dual map and the exterior product like so:

$$
a \vee b = \JJ(\JJ(a)\wedge\JJ(b))
$$

Here's a worked example of the regressive product, again using a familiar operands:

$$
\begin{aligned}
(4\ee_1 - 2\ee_{32})\vee (3\ee_{012} + \ee_{10}) &= \JJ(\JJ(4\ee_1 - 2\ee_{32})\wedge \JJ(3\ee_{012}
+ \ee_{10})) \\
&= \JJ((4\ee_{032} + 2\ee_{01}) \wedge (-3\ee_3 - \ee_{23})) \\
&= \JJ(-6\ee_{013} - 2\ee_{0123}) \\
&= -2 - 6\ee_2
\end{aligned}
$$

If you're working out the example above yourself and finding some disagreement in the signs,
remember that to use the dual map given above, the indices must match exactly. Thus, in this
example, the dual $\JJ(3\ee_{012})$ is $-\JJ(3\ee_{021}) = -3\ee_3$.

!!! tip "Exercises"

    1. If the exterior product is zero, will the regressive product be zero? Why or why not?
    2. By the same token, if the regressive product is zero, will the exterior product be zero?

## Construction

### Planes

!!! info

    [kln::plane](../../api/kln_plane)

A plane $p$ _is the manifestation of a reflection_. It is often helpful not to think of a plane as a
"set of points" in PGA as will be evident when we look at rotations and translations later. Let's
look at some simple examples first.

#### Planes through the origin

The plane represented implicitly by $x = 0$ appears in PGA simply as the vector $\ee_1$. Similarly,
the planes $y = 0$ and $3z = 0$ correspond to $\ee_2$ and $3\ee_3$ respectively. It's easy to see that
planes like $x + 3y - z = 0$ can be represented by taking linear combinations of $\ee_1$, $\ee_2$
and $\ee_3$. In this case, the plane we just created would be represented as $\ee_1 + 3\ee_2 -
\ee_3$.

```c++
// Plane x = 0
kln::plane p1{1.f, 0.f, 0.f, 0.f};

// Plane y = 0
kln::plane p2{0.f, 1.f, 0.f, 0.f};

// Plane 3z = 0
kln::plane p3{0.f, 0.f, 3.f, 0.f};

// Plane x + 3y - z = 0
kln::plane p4{1.f, 3.f, -1.f, 0.f};

// Equivalent to p4
kln::plane p5 = p1 + 3.f * p2 - p3 / 3.f;
```

#### Planes away from the origin

To shift a plane from the origin, we can add a multiple of the degenerate vector $\ee_0$. This
vector is different from the other vectors in a very important way as we will see soon.

The plane $2x + 1 = 0$ is represented as $\ee_0 + 2\ee_1$.

```c++
// Plane 2x + 1 = 0
kln::plane p6{2.f, 0.f, 0.f, 1.f};
```

Other planes like to the one above can be constructed similarly.

#### Angles between planes

The angle between planes is given by the inner product which produces the cosine of the angle
between them as you would expect. The planes must be normalized first for this to work, and the
norm of a plane $p$ can be calculated as $\sqrt{p\cdot p}$.

Suppose we have two planes $p_1 = 3\ee_0 + \ee_1 + \ee_3$ and $p_2 = \ee_0 + \ee_3$. The first plane we recognize as
$x + z - 3 = 0$, a plane parallel to the y-axis that passes through the points
$(0, y, 3)$ and $(3, y, 0)$ for any value of $y$. The second plane the plane $z + 1 = 0$ which is
parallel to the $xy$-plane and intercepts the $z$-axis one unit below the origin. We expect the
angle between these planes to be $\frac{\pi}{4}$ radians, so let's quickly verify this with the
inner product. First though, we must normalize them by dividing by the norm. This gives:

$$
\begin{aligned}
p_1 &= \frac{3\sqrt{2}}{2}\ee_0 + \frac{\sqrt{2}}{2}\ee_1 + \frac{\sqrt{2}}{2}\ee_3 \\
p_2 &= \ee_0 + \ee_3
\end{aligned}
$$

Remember that when computing the norm, because $\ee_0^2 = 0$, the $\ee_0$ component does not
participate in the computaiton. The angle between them is then computed as:

$$
\begin{aligned}
    p_1 \cdot p_2  &= \left(\frac{3\sqrt{2}}{2}\ee_0 + \frac{\sqrt{2}}{2}\ee_1 +
    \frac{\sqrt{2}}{2}\ee_3\right) \cdot (\ee_0 + \ee_3) \\
    &= \frac{\sqrt{2}}{2}\ee_3\cdot\ee_3 \\
    &= \frac{\sqrt{2}}{2}
\end{aligned}
$$

indicating that the angle between $p_1$ and $p_2$ is $\cos^{-1}{\frac{\sqrt{2}}{2}} = \frac{\pi}{4}$ radians. The Klein code to
compute the angle between planes is shown below:

```c++
float plane_angle_rad(kln::plane p1, kln::plane p2)
{
    p1.normalize(); // Normalizes p1 in place
    p2.normalize(); // Normalizes p2 in place
    return std::acos((p1 | p2).scalar());
}
```

!!! tip "Exercises"

    1. The angle calculation above is a great example to demonstrate why the degenerate element $\ee_0$ is so important. Consider what would have happened if instead $\ee_0 \cdot \ee_0 \neq 0$.
    2. Now, suppose we made the "choice" of representing planes with trivector coordinates instead of vector coordinates. Repeat the above calculation using the duals of each plane. Does the computation still work?

### Lines

!!! info

    [kln::line](../../api/kln_line)

#### Intersecting planes to create lines

The "meet" operation in $\PGA$ is defined in terms of the exterior product ($\wedge$). Given two
planes, $p_1 = 3\ee_0 + \ee_1 + \ee_3$ and $p_2 = \ee_0 + \ee_3$, we can compute the exterior
product like so:

$$
\begin{aligned}
p_1\wedge p_2 &= (3\ee_0 + \ee_1 + \ee_3) \wedge (\ee_0 + \ee_3) \\
&= 3\ee_{03} + \ee_{10} + \ee_{13} + \ee_{30} \\
&= -\ee_{01} + 2\ee_{03} - \ee_{31}
\end{aligned}
$$

How can we verify that this is correct? Well, let's first try to calculate the equation of this line
using more traditional means. In classical Euclidean geometry, the six degrees of freedom of a line
are often represented using Plücker coordinates. Let's express our planes using the implicit
equation form:

$$
\begin{aligned}
0 &= 3 + x + z \\
0 &= 1 + z
\end{aligned}
$$

The Plücker displacement is determined by the cross product of the plane normals, which in this case
is the cross product $(\mathbf{i} + \mathbf{k})\times\mathbf{k} = \mathbf{i}\times\mathbf{k} =
-\mathbf{j}$. Meanwhile, the Plücker moment of the line is given as $3\mathbf{k} - (\mathbf{i} +
\mathbf{k}) = -\mathbf{i} + 2\mathbf{k}$. Putting it together, the Plücker coordinate tuple of our
line is $(d_1:d_2:d_3:m_1:m_2:m_3) = (0: -1: 0:-1:0:2)$, in exact accordance with the calculation
above.

Observe that the coordinates associated with $\ee_{21}$, $\ee_{13}$, and $\ee_{32}$ capture the
information about the orientation of the line, while the coordinates associated with $\ee_{01}$,
$\ee_{02}$, and $\ee_{03}$ work in tandem to impart a translational element as well.

What happens if the planes are parallel? Let's try our meet between planes $-2\ee_0 + 2\ee_1$ and
$\ee_1$ to see what happens.

$$ (-2\ee_0 + 2\ee_1) \wedge \ee_1 = -2\ee_{01} $$

We ended up with a bivector all the same! Quantities that have no direction components like the
above are referred to as _ideal lines_, also known as _lines at infinity_. What's important about
concepts like lines and points at infinity is that their existence allows our algebra to have
_closure_ without needing to introduce vague notions of $\infty$. Another way to word this is to say
that planes _always_ intersect, even when parallel, and the algebra does not need to make any
special exceptions for them. Furthermore, these entities at infinity have uses! We'll see later that
rotations about ideal lines can be used to generate translations.

The code that produces the line above is the following snippet:

```c++
// plane 1: 3 + x + z = 0;
kln::plane p1{1.f, 0.f, 1.f, 3.f};

// plane 2: 1 + z = 0;
kln::plane p2{0.f, 0.f, 1.f, 1.f};

// line intersection of planes 1 and 2
kln::line intersection = p1 ^ p2;
```

!!! tip "Exercises"

    1. Match up the Plücker coordinates above with the coefficients of the meet operation above. Do you see how the element indices matter in determining the sign?
    2. Lines at infinity have 3 degrees of freedom. Is there an analogous plane at infinity? How many degrees of freedom does it have?
    3. What happens when you evaluate $p_2\wedge p_1$ instead of $p_1\wedge p_2$? Is the change justified?

### Points

!!! info

    [kln::point](../../api/kln_point)

#### Meet a line and a plane

With the same meet operation ($\wedge$) we used to intersect two planes to construct a line, we can
intersect a line and a plane to construct a point. Let's use the line $\ell = -\ee_{01} + 2\ee_{03} -
\ee_{31}$ from the previous example and the plane $\ee_1 + \ee_2$.

$$
\begin{aligned}
    (-\ee_{01} + 2\ee_{03} - \ee_{31}) \wedge (\ee_1 + \ee_2) &= -\ee_{012} + 2\ee_{031} + 2\ee_{032} - \ee_{312}\\
    &= -\ee_{123} + 2\ee_{032} + 2\ee_{031} + \ee_{021} \\
    &\rightarrow \;\; \ee_{123} - 2\ee_{032} + 2\ee_{013} - \ee_{021}
\end{aligned}
$$

In the last step, we divided the expression by $-1$ so that the weight of $\ee_{123}$ is exactly one.
For points, this weight is the homogeneous coordinate, and when it is unity, there is a direct
association between the point's Cartesian coordinates and the other trivector weights. For $\PGA$,
given a normalized point with $\ee_{123}$ weight $1$, the $x$ coordinate is the weight of the
$\ee_{032}$ trivector, the $y$ coordinate is the weight of the $\ee_{013}$ trivector, and the $z$
coordinate is the weight of the $\ee_{021}$ trivector.

!!! tip "Projective equivalence"

    In projective geometry, all geometric entities (planes, lines, points) exhibit a property known as
    projective equivalence. For any such entity $X$, the entity $aX$ represents the same entity for any
    non-zero real scalar $a$. To compare weights from one entity to another meaningfully however, we
    tend to use this projective equivalence to keep things normalized in the same way. For example, it
    doesn't make sense to compare the $\ee_{021}$ weight (the $z$-coordinate) between two points unless
    they are both normalized in the same manner.

In this case, the trivector is associated with the point at $(-2, 2, -1)$. How can we verify that
this point is the intersection we're seeking?
Let's represent our plane and line as a system of equations. Recalling that $\ell$ was
constructed as the intersection from two planes from before, this means that we can form the
following system of three equations:

$$
\begin{aligned}
0 &= 3 + x + z \\
0 &= 1 + z \\
0 &= x + y
\end{aligned}
$$

Subtracting the second equation from the first yields $x = -2$. Substituting in the third equation
yields $y = 2$. Finally, the second equation immediately gives $z = -1$.

In code, the intersection of the plane and the line to construct a point can be done as follows:

```c++
kln::line l(-1.f, 0.f, 2.f, 0.f, -1.f, 0.f);
kln::plane p{0.f, 1.f, 1.f, 0.f};
kln::point intersection = l ^ p;
```

!!! tip "Exercises"

    1. As mentioned previously, the exterior product is only anticommutative in certain situations. This isn't one of them!
       In fact, it would be somewhat concerning if the point intersection depended on the order $\ell \wedge p$ vs $p \wedge \ell$.
       Evaluate $p \wedge \ell$ for the above example and verify that you get the same result.
    2. Seeing points as trivectors might feel a bit confusing at first. This isn't so much an exercise, but a request for
       you, the reader, to reserve any doubts that the representation makes sense until we look at the geometric product
       in view of symmetric actions.
    3. Here, we met a line and a plane to get the point of intersection point. Three planes can be met with the same operator $\wedge$
       to produce the intersection point as well. Can you see why?

### Joins

In the constructions above, we used the exterior product to _meet_ entities to elegantly construct
intersections. The $\wedge$ operator is a grade-increasing operation that allowed us to go from
planes to lines to points. What about going the other direction? Well, there's a handy tool for
that! Earlier, we learned about the [Regressive Product](#regressive-product), defined _in terms_ of
the exterior product on the duals of the operands. As a result, it should be evident that the
regressive product is a _grade decreasing_ operation that allows us to _join_ entities to go in the
other direction, from points to lines to planes.

Sure enough, this works exactly as you'd expect. Two points can be joined to construct a line, and
a line and a point (or equivalently three points) can be joined to create a plane. Here's a code
snippet doing exactly this.

```c++
kln::point p1{x1, y1, z1};
kln::point p2{x2, y2, z2};

kln::line p1_to_p2 = p1 & p2;

kln::point p3{x3, y3, z3};

/// Equivalent to p1 & p2 & p3;
kln::plane p1_p2_p3 = p1_to_p2 & p3;
```

!!! tip "Exercises"

    1. Try to perform the computation above by hand given sufficiently simple initialization values of the three points.
       Do the results agree with what you expect?
    2. Explain why the Poincaré dual map we introduced earlier needed to be an involution for our join to work so
       conveniently.

## Reflections

So far, we've seen how the exterior product and regressive product can be used to construct planes,
lines, and points. We also saw how the symmetric inner product can be used to measure angles between
planes. However, we haven't yet encountered the usage of the geometric product, and this is where
we'll fully appreciate the decision of using the representation we have.

### Reflection through a plane

A reflection of an entity $X$ through a plane $p$ is given by $pXp$. This is true regardless of
whether $X$ is another plane, a line, or a point!

!!! danger "ARTICLE CURRENTLY BEING DRAFTED"