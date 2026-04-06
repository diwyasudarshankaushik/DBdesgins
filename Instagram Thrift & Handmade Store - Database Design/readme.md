# Instagram Thrift & Handmade Store - Database Design
This repository contains the Entity-Relationship (ER) design for a growing Instagram-based retail business. The model is designed to transition a manual "DM-to-order" workflow into a structured system that tracks unique inventory, customer history, and fulfillment status.

## 📌 Business Logic & Challenges
The primary challenge of this design was handling two distinct product types:

1. Thrifted Items: Typically one-of-a-kind (Quantity = 1) with specific attributes like "Condition" (e.g., Gently Used, Vintage).

2. Handmade Items: Can be produced in batches (Quantity > 1) and may have standard variations.

<!-- ## 🏗️ Entities and Attributes

 **1.Products & Inventory** 

 • **Products**: The central catalog.

    • product_id (PK), name, description, price, category, product_type (Enum: Thrift/Handmade).

 • Product_Details: Stores specific metadata.

    • detail_id (PK), product_id (FK), size, color, condition_notes.

> Note : Condition is crucial for thrifted items but can be marked "New" for handmade.

**Inventory:**

inventory_id (PK), product_id (FK), stock_quantity.

For Thrifted items, this defaults to 1 and moves to 0 upon sale.

2. Customers & Orders

Customers: Stores buyer profiles.

customer_id (PK), ig_handle, whatsapp_number, email, default_shipping_address.

Orders: The header for a transaction.

order_id (PK), customer_id (FK), order_date, total_amount, order_status (Pending, Packed, Shipped, Delivered).

Order_Items (Junction Table): Handles the Many-to-Many relationship between Orders and Products.

order_item_id (PK), order_id (FK), product_id (FK), quantity, price_at_sale.

3. Fulfillment

**•Payments:**

``` payment_id (PK), order_id (FK), payment_method (UPI, Bank Transfer), transaction_id, payment_status.```

**•Shipping:**

``` shipping_id (PK), order_id (FK), tracking_number, courier_partner, estimated_delivery. ``` -->


## 📝 Business Operational Flow with example

1. **Inquiry** -  Check `Products` & `Inventory`: Verify if the Handmade Beanie is in stock and if the unique Vintage Denim Jacket is still available.
2. **Customer Setup** -  Insert into `Customers`: Create a new profile or retrieve an existing one using their `ig_handle`.
3. **Order Creation** - Insert into `Orders`: Create an order record with a Pending status and the current timestamp.
4. **Item Mapping** - insert into `Order_Items`: Add two rows—one for the Jacket, one for the bag.
5. **Payment** - Insert into `Payments`: Record the `transaction_id` and update status to `Completed`**.
6. **Stock Update** - Update `Inventory`: Reduce the Handmade Beanie stock by 1; set the thrifted Vintage Denim Jacket stock to 0.
7. **Shipping** - Insert into `Shipping`: Add the `tracking_number` and set status to `Shipped`.


> Imagine a customer named Diwya buying two items. Here is how your tables would look:

**Step A: The Products (Initial State)**

- Product ID 1: "Vintage Denim Jacket" | category: Thrift | Stock: 1 | Price: ₹1200\
- Product ID 2: "Handmade Beanie" | category: Handmade | Stock: 10 | Price: ₹500

**Step B: The Customer**

- Customer ID 5: Diwya | IG: @Diwya_vibe | Address: Sector 12, Jaipur

**Step C: The Transaction (The Order)**

- Orders Table: Order #101 | Customer ID: 5 | Total: ₹1700

**Step D: The Link (Order_Items)**

- Row 1: Order #101 | Product: 1 (Jacket) | Qty: 1 | Price: ₹1200\
- Row 2: Order #101 | Product: 2 (Beanie) | Qty: 1 | Price: ₹500

**Step E: The Result**

1. Inventory Change: Jacket stock becomes 0 (Marked as Sold). Beanie stock becomes 9.\
2.  Shipping Record: Order #101 | Carrier: "Delhivery" | Tracking: XYZ123 | Status: Shipped.


## 🔗 Relationships & Cardinality
 
- **Customer ↔ Order (1:N):** One customer can place many orders, but an order belongs to one customer.

- **Order ↔ Product (N:M):** An order can contain multiple products, and a product (especially handmade) can appear in multiple orders. This is resolved via the Order_Items junction table.

- **Product ↔ Inventory (1:1):** Each product entry tracks its own stock level.

- **Order ↔ Payment/Shipping (1:1):** Each unique order has one specific payment record and one shipping tracking record.

## 🛠️ Design Decisions

- **Handling Thrift vs. Handmade:** Instead of creating two separate tables, I used a product_type attribute. This keeps the Order_Items logic simple while allowing the frontend to filter or display "Condition" only if the type is 'Thrift'.

- **Normalization:** I separated Payment and Shipping from the main Orders table. This prevents the Orders table from becoming "fat" and allows for multiple payment attempts or shipping updates without duplicating order data.

- **Price Snapshot:** The Order_Items table includes price_at_sale. This is a database best practice to ensure that if a creator changes a product's price later, the historical record of what the customer actually paid remains accurate.

## 🚀 How to Use
1. Open the provided ER Diagram image alongside this documentation.

2. Follow the Primary Key (PK) and Foreign Key (FK) lines to see how data flows from a customer's DM to a completed delivery.
