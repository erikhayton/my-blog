---
layout: post
title: Export your PrestaShop orders without module
---
With a default install of PrestaShop (any version), you cannot export all your orders and clients data to a file. You can either export your orders (but do not have any reference about the client) or your clients (but can't see his orders). That sounds a bit crazy but this is the choice they made.

But what if you want to send an email to your last 100 clients?  

The only solutions are (were ^^):
<ul>
	<li>Install a paid module (such as <a href="http://addons.prestashop.com/fr/17596-orders-csv-excel-export.html" target="_blank">http://addons.prestashop.com/fr/17596-orders-csv-excel-export.html</a>)</li>
	<li>Develop your own module</li>
	<li>Make it manually</li>
</ul>

If you do not want to pay or don't have time or skills to create your own module, then there is another solution: use a plain-old SQL query :)

Just open your MySQL client (such as PHPMyAdmin, MySQL Workbench or whatever) and open your PrestaShop database.

Then execute the following query:

​`​``SELECT o.id_order, o.id_customer, o.date_add, c.firstname, c.lastname, c.email FROM ps_orders o LEFT JOIN ps_customer c ON o.id_customer = c.id_customer WHERE o.current_state = 5 ORDER BY o.date_add DESC`

I know this is a simple one, but I saw so much clients being stuck to export their clients list ​that I thought it was a good idea to share it.

You can now easily export the result!

PS: This query only returns the delivered orders. If you want to get all orders or use another filter, you can do so using the table on this page: <a href="http://doc.prestashop.com/display/PS16/Statuses" target="_blank">http://doc.prestashop.com/display/PS16/Statuses</a>.