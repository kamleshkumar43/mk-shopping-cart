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

## Project status

The repository currently contains package metadata and this README, but no shopping-cart implementation or tests. The command-line entry point is declared in the project metadata; it will be usable once the corresponding application package is implemented.

The project metadata requires Python 3.14 or newer and currently lists no external dependencies.


