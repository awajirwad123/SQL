# 30. Normalization (1NF, 2NF, 3NF)

Q: Purpose?
A: Eliminate redundancy, update anomalies, and ensure data integrity via structured decomposition.

1NF: Atomic column values (no repeating groups / arrays). Example violation: phone1, phone2 columns; fix with child phone table.

2NF: 1NF + every non-key attribute fully depends on the whole primary key (applies to composite keys). Example violation: (order_id, product_id) key with product_name depending only on product_id.

3NF: 2NF + no transitive dependency (non-key depending on another non-key). Example violation: customers(id, zip, city, state) where city/state depend on zip.

Example progression:
Raw: orders(order_id, product_id, product_name, category_name) → Move product_name, category_name to products(product_id, product_name, category_name).

Pitfall: Over-normalizing for read-heavy analytics causing too many joins.
