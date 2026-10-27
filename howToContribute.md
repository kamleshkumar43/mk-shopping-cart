# Beginner's Guide to Building the Shopping Cart

This guide turns the planned features in `README.md` into a small set of Python classes and methods. The repository does not contain the application code yet, so use these steps as an implementation plan.

## First: classes, objects, and methods

- A **class** is a recipe for a kind of thing. For example, `Product` describes the information every product should have.
- An **object** is one actual thing made from a class. A red mug priced at 12.50 could be one `Product` object.
- An **attribute** is information stored on an object, such as a product's name or price.
- A **method** is an action an object can perform. For example, a cart's `add_product` method adds a product to that cart.

In Python, define a class with `class Product:` and create an object by calling it, such as `Product("Mug", 12.50)`. The `__init__` method sets up the new object's attributes.

## Main classes

### `Product`

Represents one item that can be sold.

Store a product ID, name, description, and price. Keep the price as a decimal amount rather than a floating-point number so money calculations are predictable.

Methods to create:

- `__init__(product_id, name, description, price)`: save the product details and reject a negative price or missing name.
- `update_price(new_price)`: change the price after checking that it is not negative.

### `ProductCatalog`

Keeps the products that customers can browse. A catalog can be a simple list or dictionary at first.

Methods to create:

- `add_product(product)`: add a product to the catalog.
- `find_by_id(product_id)`: return the matching product, or report that it was not found.
- `list_products()`: return all available products for display.

### `CartItem`

Represents one product and its quantity inside a cart. It is different from `Product`: the product describes what is sold, while the cart item describes how many the customer wants right now.

Store a reference to a `Product` object and a positive quantity.

Methods to create:

- `__init__(product, quantity)`: save the product and quantity; reject a quantity below one.
- `line_total()`: return the product's current price multiplied by the quantity.
- `change_quantity(new_quantity)`: set a valid quantity. Removing the item should be handled by the cart.

### `Discount`

Represents one promotion that a customer may apply to a cart.

Store a promo code, discount type (percentage or fixed amount), value, and any initial rules, such as a minimum subtotal or eligible product IDs.

Methods to create:

- `is_valid_for(cart)`: check that the promotion is active and the cart meets its rules.
- `calculate_savings(eligible_amount)`: calculate the percentage or fixed-amount saving, never returning more than the eligible amount.

The first version should allow no more than one discount on a cart. More advanced rules, such as combining codes or limiting uses per customer, can be added later.

### `DiscountCatalog`

Finds a promotion when the customer enters its code.

Methods to create:

- `add_discount(discount)`: store a discount that can be offered.
- `find_by_code(code)`: return the matching discount, or report that the code is unknown.

### `Cart`

Belongs to one customer and holds that customer's current `CartItem` objects. It may also hold one applied `Discount` object.

Methods to create:

- `add_product(product, quantity=1)`: add a new cart item, or increase the quantity if that product is already in the cart.
- `remove_product(product_id)`: remove the matching item from the cart.
- `change_quantity(product_id, new_quantity)`: update an item's quantity; reject zero or negative quantities and use `remove_product` to remove it.
- `subtotal()`: add the line totals for all cart items before discounts.
- `apply_discount(discount)`: check the discount rules, then store it on the cart. Replace or reject an existing discount; do not stack discounts in the first version.
- `discount_eligible_subtotal(discount)`: add the line totals for products that qualify for this discount. If the discount applies to the whole cart, use the cart subtotal.
- `discount_savings()`: return zero when no discount is applied, otherwise ask the discount to calculate savings from the eligible subtotal.
- `total()`: return the subtotal minus the savings, never less than zero.
- `checkout(customer)`: create an order from the cart, then clear the cart only after order creation succeeds.
- `clear()`: remove all items and the applied discount after a successful checkout.

### `Customer`

Represents the person shopping. For the first version, store a customer ID, a name, and one cart.

Methods to create:

- `__init__(customer_id, name)`: save the customer's details and create an empty cart for them.
- `checkout()`: ask the cart to check out this customer and return the resulting order.

### `OrderItem`

Represents one purchased product inside an order. Unlike a `CartItem`, it must preserve the details used at checkout, even if the product's name or price changes later.

Store a product ID, product name, unit price, and purchased quantity as values copied at checkout.

Methods to create:

- `line_total()`: return the saved unit price multiplied by the saved quantity.

### `Order`

Represents a completed checkout. Store an order ID, the customer's ID, a list of `OrderItem` objects, the discount code and savings (if used), and the final total at checkout.

Methods to create:

- `from_cart(customer, cart)`: build an order by copying cart item details and the applied discount. This is a class method because it creates a new order from existing objects.
- `total()`: return the total saved when the order was created. Do not recalculate old orders from today's product prices or discount rules.
- `summary()`: provide the order details for display to the customer.

## How the objects fit together

```text
+------------------+       owns       +------------------+
|     Customer     | 1 ------------ 1 |       Cart       |
+------------------+                  +------------------+
											   |
											   | contains many
											   v
									  +------------------+
									  |     CartItem     | ---- refers to ----> Product
									  +------------------+
											   |
											   | may use zero or one
											   v
									  +------------------+
									  |     Discount     |
									  +------------------+

At checkout:
CartItem details and discount savings are copied into an Order
Order contains many OrderItems; each OrderItem keeps a price snapshot
```

The arrows show which objects refer to or contain other objects. A cart item points to a product, but an order item saves a copy of the product's name and price so order history does not change unexpectedly.

## Build it in stages

### Stage 1: Create products

Implement `Product` and `ProductCatalog` first. Add a few sample products and check that listing and looking up products works. This gives the cart something to contain.

### Stage 2: Build a cart

Implement `CartItem`, then `Cart`. Start with adding products, changing quantities, removing products, and calculating the subtotal. Try adding the same product twice; the cart should have one line whose quantity increases.

### Stage 3: Add customers

Implement `Customer` so each customer gets their own empty cart. Check that changing one customer's cart does not change another customer's cart.

### Stage 4: Add discounts

Implement `Discount` and `DiscountCatalog`. Look up a code, check its rules against the cart, and calculate savings. Test both a percentage discount and a fixed-amount discount. Confirm the savings never exceed the eligible amount and that only one code is applied at a time.

### Stage 5: Check out and save an order

Implement `OrderItem` and `Order`. `Cart.checkout(customer)` uses `Order.from_cart(customer, cart)` to copy the cart's product names, prices, quantities, and discount details into the order. Only clear the cart after order creation succeeds. Then change a product's price and confirm the old order's total stays the same.

### Stage 6: Connect the workflow

Use the project's `main()` entry point to connect the classes. A simple first workflow can list products, let a customer add or remove items, show the subtotal, accept an optional promo code, show the final amount, and create an order when the customer checks out.

## Example shopping flow

1. The catalog displays products.
2. The customer asks the cart to add products. The cart creates or updates `CartItem` objects.
3. The cart calculates its subtotal from its cart items.
4. The customer enters a promo code. The discount catalog finds it, and the cart checks whether it can be used.
5. The cart displays the savings and amount due.
6. The customer checks out. The cart creates an order with saved copies of the item and discount details.
7. After the order is created successfully, the cart is cleared.

## A good way to test each stage

Test one small behavior at a time: a product rejects a negative price, a cart line total is correct, adding the same product increases its quantity, an unknown promo code is rejected, a discount cannot make the total negative, and an order total stays unchanged after a product price changes. These checks make problems easier to find before the full workflow is connected.

Payment processing, taxes, shipping, persistent storage, customer sign-in, and advanced discount rules are outside this first implementation plan, as noted in `README.md`.
