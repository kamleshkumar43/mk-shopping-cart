# Shopping Cart

## About

Shopping Cart is intended to let a customer collect products, change quantities, review the cost, and place an order. This repository is currently a project scaffold: the application code has not been added yet. The features and object relationships below describe the planned behavior, not functionality that is already available.

## Planned functionality

- Show a list of products with a name, description, and price.
- Add a product to the customer's cart.
- Increase or decrease the quantity of a product in the cart.
- Remove a product from the cart.
- Show each cart line's price and quantity, plus the cart subtotal.
- Apply one promo code to a cart and show the savings and updated amount due.
- Check that a discount is active and that its rules are met, such as a minimum spend or eligible products.
- Do not let a discount reduce the amount due below zero.
- Keep the cart available while the customer continues shopping.
- Review the cart before placing an order.
- Create an order from the cart at checkout. The order keeps a record of the product names, prices, quantities, and discount at the time it was placed.
- Show the customer the order and its total after checkout.

Payment processing, shipping, tax calculation, inventory management, and customer account sign-in are not defined yet. The discount rules below are an initial proposal.

## How the objects relate

The customer owns a cart. The cart holds cart items, and each cart item points to one product. A product can appear in many carts. At checkout, the cart's contents are copied into an order so later cart or product changes do not rewrite the order history.

A discount describes a promotion, such as a percentage off or a fixed amount off. The cart can have one discount applied if the code is valid and its rules are met. At checkout, the order records the discount details and the amount saved so its total stays accurate if the discount later changes or expires.

```text
+------------------+       owns       +------------------+
|     Customer     | 1 ------------ 1 |       Cart       |
+------------------+                  +------------------+
											   |
											   | contains
											   | 1
											   |
											   | many
									  +------------------+
									  |     CartItem     |
									  +------------------+
											   |
											   | refers to
											   | many-to-one
											   v
									  +------------------+
									  |     Product      |
									  +------------------+
```

At checkout, the cart's items become order items. An order item stores a snapshot of the product name and price, along with the purchased quantity.

```text
+------------------+       places       +------------------+
|     Customer     | 1 -------------- * |      Order       |
+------------------+                    +------------------+
												|
												| contains
												| 1
												|
												| many
									   +------------------+
									   |    OrderItem     |
									   +------------------+
												|
												| records the product
												| and price at checkout
												v
									   +------------------+
									   |     Product      |
									   +------------------+
```

`*` means "many." A customer may place multiple orders. Each order has one or more order items. The order item keeps its own name and price snapshot; its product reference is useful for identifying the original product, not for recalculating the historical order total.

The discount is optional; a cart can have no discount. The first version allows at most one applied discount per cart.

```text
+------------------+  may apply 0 or 1  +------------------+
|       Cart       | -----------------> |     Discount     |
+------------------+                    +------------------+
	|
	| checkout records the discount and savings
	v
+------------------+
|      Order       |
+------------------+
```

## Planned calculations

- Cart line total = product price multiplied by quantity.
- Cart subtotal = the sum of all cart line totals.
- Percentage discount = eligible amount multiplied by the discount rate.
- Fixed-amount discount = the discount amount, capped at the eligible amount.
- Amount due = subtotal minus discount. It cannot be less than zero.
- Order total = the amount due recorded at checkout.

Taxes, shipping costs, and currency rules will need to be decided before the final total can include them. More advanced discount rules, such as combining codes or limiting uses per customer, are not defined yet.

## Proposed project structure

The following is a suggested layout for implementing the application. These files and folders have not been created yet.

```text
mk-shopping-cart/
|-- pyproject.toml
|-- README.md
|-- howToContribute.md
|-- src/
|   `-- mk_shopping_cart/
|       |-- __init__.py
|       |-- cli.py
|       |-- product.py
|       |-- product_catalog.py
|       |-- cart_item.py
|       |-- discount.py
|       |-- discount_catalog.py
|       |-- cart.py
|       |-- customer.py
|       |-- order_item.py
|       `-- order.py
`-- tests/
	|-- test_product.py
	|-- test_product_catalog.py
	|-- test_cart.py
	|-- test_discount.py
	`-- test_order.py
```

### Application files

- `src/mk_shopping_cart/__init__.py`: marks this folder as a Python package and exposes the `main()` function from `cli.py`. The `main()` export matches the command configured in `pyproject.toml`.
- `src/mk_shopping_cart/cli.py`: contains `main()` and handles the simple text-based interaction with the shopper. It connects the other classes but should leave product, cart, discount, and order rules to those classes.
- `src/mk_shopping_cart/product.py`: defines the `Product` class and validates product details such as name and price.
- `src/mk_shopping_cart/product_catalog.py`: defines `ProductCatalog`, which stores products and supports listing them or finding one by ID.
- `src/mk_shopping_cart/cart_item.py`: defines `CartItem`, which pairs one product with a quantity and calculates that line's total.
- `src/mk_shopping_cart/discount.py`: defines `Discount`, which checks promotion rules and calculates percentage or fixed-amount savings.
- `src/mk_shopping_cart/discount_catalog.py`: defines `DiscountCatalog`, which finds a discount from the code entered by the shopper.
- `src/mk_shopping_cart/cart.py`: defines `Cart`, which adds and removes items, changes quantities, calculates totals, applies a discount, and starts checkout.
- `src/mk_shopping_cart/customer.py`: defines `Customer` and gives each customer their own cart.
- `src/mk_shopping_cart/order_item.py`: defines `OrderItem`, which saves the product name, price, and quantity used when an order is placed.
- `src/mk_shopping_cart/order.py`: defines `Order`, which creates and displays a saved record of a completed checkout.

### Test files

Tests are small checks that confirm the classes behave as expected. Keep them separate from the application code.

- `tests/test_product.py`: checks product details and price validation.
- `tests/test_product_catalog.py`: checks adding, listing, and finding products.
- `tests/test_cart.py`: checks adding products, quantities, removals, and cart totals.
- `tests/test_discount.py`: checks discount codes, eligibility, and savings limits.
- `tests/test_order.py`: checks checkout, saved prices, and order totals.

### Project and guide files

- `pyproject.toml`: stores package information, the Python version requirement, dependencies, and the `mk-shopping-cart` command. Its configured command calls `mk_shopping_cart:main`, so `__init__.py` must expose `main()`.
- `README.md`: explains what the shopping cart is intended to do, how its objects relate, and the proposed source layout.
- `howToContribute.md`: gives new contributors a step-by-step explanation of classes, methods, and how to build the project.

## Project status

The repository currently contains package metadata and this README, but no shopping-cart implementation or tests. The command-line entry point is declared in the project metadata; it will be usable once the corresponding application package is implemented.

The project metadata requires Python 3.14 or newer and currently lists no external dependencies.


