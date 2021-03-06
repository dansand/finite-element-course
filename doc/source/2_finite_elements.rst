.. default-role:: math

.. _secfinitelement:

Constructing finite elements
============================

At the core of the finite element method is the representation of
finite-dimensional function spaces over elements. This concept was
formalised by :cite:`Ciarlet2002`:

.. _def-ciarlet:

.. definition:: 

   A *finite element* is a triple `(K, P, N)` in which `K` is a cell,
   `P` is a space of functions `K\rightarrow\mathbb{R}^n` and `N`, the
   set of *nodes*, is a basis for `P^*`, the `dual space
   <http://mathworld.wolfram.com/DualVectorSpace.html>`_ to `P`.

Note that this definition includes a basis for `P^*`, but not a
basis for `P`. It turns out to be most convenient to specify the set
of nodes for an element, and then derive an appropriate basis for
`P` from that. In particular:

.. definition::

   Let `N = \{n_j\}` be a basis for `P^*`.  A *nodal
   basis*, `\{\phi_i\}` for `P` is a basis for `P`
   with the property that `n_j(\phi_i) = \delta_{ij}`.

A worked example
----------------

To illustrate the construction of a nodal basis, let's consider the
linear polynomials on a triangle. We first need to define our
reference cell. The obvious choice is the triangle with vertices
`\{(0,0), (1,0), (0,1)\}` 

Functions in this space have the form `a + bx + cy`. So the
function space has three unknown parameters, and its basis (and dual
basis) will therefore have three members. In order to ensure the correct
continuity between elements, the dual basis we need to use is the
evaluation of the function at each of the cell vertices. That is:

.. math::
  :label:

  n_0(f) = f\left((0,0)\right)

  n_1(f) = f\left((1,0)\right)

  n_2(f) = f\left((0,1)\right)

We know that `\phi_i` has the form `a + bx + cy` so now we can
use the definition of the nodal basis to determine the unknown
coefficients:

.. math::
  :label:

  \begin{pmatrix}
  n_0(\phi_i)\\
  n_1(\phi_i)\\
  n_2(\phi_i)
  \end{pmatrix}
  =
  \begin{pmatrix}
  \delta_{i,0}\\
  \delta_{i,1}\\
  \delta_{i,2}
  \end{pmatrix}

So for `\phi_0` we have:

.. math::
  :label: phimat

  \begin{bmatrix}
  1 & 0 & 0\\
  1 & 1 & 0\\
  1 & 0 & 1\\
  \end{bmatrix}
  \begin{bmatrix}
  a\\
  b\\
  c\\
  \end{bmatrix}
  = 
  \begin{bmatrix}
  1 \\
  0 \\
  0
  \end{bmatrix}

Which has solution `\phi_0 = 1 - x - y`. By a similar process,
we can establish that the full basis is given by:

.. math::
   :label:

   \phi_0 = 1 - x - y
   
   \phi_1 = x

   \phi_2 = y


Types of node
-------------

We have just encountered nodes given by the evaluation of the function
at a given point. Other forms of functional are also suitable for use
as finite element nodes. Examples include the integral of the function
over the cell or some sub-entity and the evaluation of the gradient
of the function at some point. For some vector-valued function spaces,
the nodes may be given by the evaluation of the components of the
function normal or tangent to the boundary of the cell at some point.

In this course we will only consider point evaluation nodes. The implementation of several other forms of node are covered in :cite:`Kirby2004`.

The Lagrange element nodes
--------------------------

The number of coefficients of a degree `p` polynomial in `d`
dimensions is given by `\begin{pmatrix}p+d\\ d\end{pmatrix}`. The
simplest set of nodes which we can employ is simply to place these
nodes in a regular grid over the reference cell. Given the classical
relationship between binomial coefficients and `Pascal's triangle
<http://mathworld.wolfram.com/PascalsTriangle.html>`_ (and between
trinomial coefficients and Pascal's pyramid), it is unsurprising that
this produces the correct number of nodes.

The set of equally spaced points of degree `p` on the triangle is:

.. math::
   :label: lattice

   \left\{\left(\frac{i}{p}, \frac{j}{p}\right)\middle| 0 \leq i+j \leq p\right\}
  
The finite elements with this set of nodes are called the *equispaced
Lagrange* elements and are the most commonly used elements for
relatively low order computations. 

.. note::

   At higher order the equispaced Lagrange basis is poorly conditioned
   and creates unwanted oscillations in the solutions. However for
   this course Lagrange elements will be sufficient.

.. _ex-lagrange-points:

.. exercise::
   
   Use :eq:`lattice` to implement
   :func:`~fe_utils.finite_elements.lagrange_points`. Make sure your
   algorithm also works for one-dimensional elements. Some basic tests
   for your code are to be found in
   ``test/test_lagrange_points.py``. You can also test your lagrange
   points on the triangle by running:: 

     python test/plot_lagrange_points.py degree
   
   From the Enthought Canopy Python command line, you would instead write::

     run test/plot_lagrange_points.py degree

   Where degree is the degree of the points to plot.

.. hint::
   
   The lagrange points can be generated very simply using `list
   comprehensions
   <https://docs.python.org/2/tutorial/datastructures.html#list-comprehensions>`_.

.. _sec-vandermonde:

Solving for basis functions
---------------------------

The matrix in :eq:`phimat` is a *generalised Vandermonde* [#vandermonde]_
matrix . Given a list of points `(x_i,y_i) \in \mathbb{R}^2, 0\leq i< m`
the corresponding degree `n` generalised Vandermonde matrix is given by:

.. math::
    :label: Vandermonde

    \mathrm{V} = 
    \begin{bmatrix}
    1 & x_0 & y_0 & x_0^2 & x_0y_0 & y_0^2 & \ldots & x_0^n & x_0^{n-1}y_0 & \ldots & x_0y_0^{n-1} & y_0^n \\
    1 & x_1 & y_1 & x_1^2 & x_1y_1 & y_1^2 & \ldots & x_1^n & x_1^{n-1}y_1 & \ldots & x_1y_1^{n-1} & y_1^n \\
    \vdots \\
    1 & x_m & y_m & x_m^2 & x_my_m & y_m^2 & \ldots & x_m^n & x_m^{n-1}y_m & \ldots & x_my_m^{n-1} & y_m^n \\
    \end{bmatrix}

If we construct the Vandermonde matrix for the nodes of a finite
element, then the equation for the complete set of basis function
polynomial coefficients is:

.. math::
   :label: vdm-equation

   \mathrm{V}\mathrm{C} = \mathrm{I}

where the `j`-th column of `C` contains the polynomial coefficients of
the basis function corresponding to the `j`-th node. For
:eq:`vdm-equation` to be well-posed, there must be a number of nodes
equal to the number of coefficients of a degree `n` polynomial. If
this is the case, then it follows immediately that:

.. math::
   :label:
   
   \mathrm{C} = \mathrm{V}^{-1}

The same process applies to the construction of basis functions for
elements in one or three dimensions, except that the Vandermonde
matrix must be modified to exclude powers of `y` (in one dimension) or
to include powers of `z`. 

.. note::

   The power series basis for polynomial spaces employed here becomes
   increasingly ill-conditioned at higher order, so it may be
   advantageous to employ a different basis in the construction of the
   Vandermonde matrix. See :cite:`Kirby2004` for an example.

.. _ex-vandermonde:

.. exercise::

   Use :eq:`Vandermonde` to implement
   :func:`~fe_utils.finite_elements.vandermonde_matrix`. Think
   carefully about how to loop over each row to construct the correct
   powers of `x` and `y`. For the purposes of this exercise you should
   ignore the ``grad`` argument.

   Tests for this function are in ``test/test_vandermonde_matrix.py``

.. hint::

   You can use numpy array operations to construct whole columns of
   the matrix at once. 
 

Implementing finite elements in Python
--------------------------------------

The :ref:`Ciarlet triple <def-ciarlet>` `(K, P, N)` also provides a
good abstraction for the implementation of software objects
corresponding to finite elements. In our case `K` will be a
:class:`~fe_utils.reference_elements.ReferenceCell`. In this course we
will only implement finite element spaces consisting of complete
polynomial spaces so we will specify `P` by providing the maximum
degree of the polynomials in the space. Since we will only deal with
point evaluation nodes, we can represent `N` by a series of points at
which the evaluation should occur.

.. _ex-finite-element:

.. exercise::

   Implement the rest of the
   :class:`~fe_utils.finite_elements.FiniteElement` :meth:`__init__`
   method. You should construct a Vandermonde matrix for the nodes and
   invert it to create the basis function coefs. Store these as
   ``self.basis_coefs``. 

   Some basic tests of your implementation are in
   ``test/test_init_finite_element.py``.

.. hint::
   The :func:`numpy.linalg.inv` function may be
   used to invert the matrix.


Implementing the Lagrange Elements
----------------------------------

The :class:`~fe_utils.finite_elements.FiniteElement` class implements
a general finite element object assuming we have provided the cell,
polynomial, degree and nodes. The
:class:`~fe_utils.finite_elements.LagrangeElement` class is a
`subclass
<https://docs.python.org/2/tutorial/classes.html#inheritance>`_ of
:class:`~fe_utils.finite_elements.FiniteElement` which will implement
the particular case of the equispaced Lagrange elements.

.. _ex-lagrange-element:

.. exercise::

   Implement the :meth:`__init__` method of
   :class:`~fe_utils.finite_elements.LagrangeElement`. Use
   :func:`~fe_utils.finite_elements.lagrange_points` to obtain the
   nodes. For the purpose of this exercise, you may ignore the
   ``entity_nodes`` argument.

   **After** you have implemented
   :meth:`~fe_utils.finite_elements.FiniteElement.tabulate` in the
   next exercise, you can use
   ``plot_lagrange_basis_functions.py`` to visualise your
   Lagrange basis functions.

Tabulating basis functions
--------------------------

A core operation in the finite element method is integrating
expressions involving functions in finite element spaces. This is
usually accomplished using :doc:`numerical quadrature
<1_quadrature>`. This means that we need to be able to evaluate the
basis functions at a set of quadrature points. The operation of
evaluating a set of basis functions at a set of points is called
*tabulation*.

.. _ex-tabulate:

.. exercise::
   
   Implement :meth:`~fe_utils.finite_elements.FiniteElement.tabulate`.
   You can use a Vandermonde matrix to evaluate the polynomial terms
   and take the matrix product of this with the basis function
   coefficients. The method should have at most two executable
   lines. For the purposes of this exercise, ignore the ``grad``
   argument.

   The test file ``test/test_tabulate.py`` checks that tabulating the
   nodes of a finite element produces the identity matrix.

Gradients of basis functions
----------------------------

A function `f` defined over a single finite element with basis
`\{\phi_i\}` is represented by a weighted sum of that basis:

.. math::

   f = \sum_i f_i\phi_i

In order to be able to represent and solve PDEs, we will naturally
also have terms incorporating derivatives. Since the coefficients
`f_i` are spatially constant, derivative operators pass through to
apply to the basis functions:

.. math::

   \nabla f  = \sum_i f_i\nabla\phi_i

This means that we will need to be able to evaluate the gradient of
the basis functions at quadrature points. 

.. exercise::
   
   Extend :meth:`~fe_utils.finite_elements.vandermonde_matrix` so that
   setting ``grad`` to ``True`` produces a rank 3 generalised
   Vandermonde tensor whose indices represent points, gradient
   component and basis function respectively. That is, each entry of
   `V` is replaced by a vector of the gradient of that polynomial
   term. For example, the entry `x^2y^3` would be replaced by the
   vector `[ 2xy^3, 3x^2y^2 ]`.

   The ``test/test_vandermonde_matrix_grad.py`` file has tests of this
   extension. You should also ensure that you still pass
   ``test/test_vandermonde_matrix.py``.

.. hint:: 

   The :meth:`~numpy.ndarray.transpose` method of numpy arrays enables
   generalised transposes swapping any dimensions.

.. exercise::

   Extend :meth:`~fe_utils.finite_elements.FiniteElement.tabulate` to
   pass the ``grad`` argument through to
   :meth:`~fe_utils.finite_elements.vandermonde_matrix`. Then
   generalise the matrix product in
   :meth:`~fe_utils.finite_elements.FiniteElement.tabulate` so that
   the result of this function (when ``grad`` is true) is a rank 3
   tensor:

   .. math::

      \mathrm{T}_{ijk} = \nabla(\phi_j(X_i))\cdot \mathbf{e}_k

   where `\mathbf{e}_0\ldots\mathbf{e}_{\dim -1}` is the coordinate
   basis on the reference cell.

   The ``test/test_tabulate_grad.py`` script tests this
   extension. Once again, make sure you still pass
   ``test/test_tabulate.py``

.. hint::

   The :func:`numpy.einsum` function implements generalised tensor
   contractions using `Einstein summation notation
   <http://mathworld.wolfram.com/EinsteinSummation.html>`_. For
   example, ``A = numpy.einsum("ijk,jl->ilk", T, C)`` is equivalent to
   `A_{ilk} = \sum_j T_{ijk} C_{jl}`.

Interpolating functions to the finite element nodes
---------------------------------------------------

Recall once again that a function can be represented on a single finite element as:

.. math::

   f = \sum_i f_i\phi_i

Since `\{\phi_i\}` is a nodal basis, it follows immediately that:

.. math::
   
   f_i = \phi_i^*(f)

where `\phi_i^*` is the node associated with the basis function
`\phi_i`. Since we are only interested in nodes which are the point
evaluation of their function input, we know that:

.. math::

   f_i = f(X_i)

where `X_i` is the point associated with the `i`-th node.

.. _ex-interpolate:

.. exercise::

   Implement :meth:`~fe_utils.finite_elements.FiniteElement.interpolate`.

   Once you have done this, you can use the script provided to plot
   functions of your choice interpolated onto any of the finite
   elements you can make::

     python test/plot_interpolate_lagrange.py "sin(2*pi*x[0])" 2 5

   From the Enthought Canopy Python command line, you would instead write::

     run test/plot_interpolate_lagrange.py "sin(2*pi*x[0])" 2 5

   .. hint::

      You can find help on the arguments to this function with::
      
        python test/plot_interpolate_lagrange.py -h

.. rubric:: Footnotes

.. [#vandermonde] A `Vandermonde
                  matrix <http://mathworld.wolfram.com/VandermondeMatrix.html>`_
                  is the one-dimensional case of the generalised Vandermonde matrix.
