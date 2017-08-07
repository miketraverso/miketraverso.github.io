---
layout: post
title:  "Hiding empty table cells in UITableViews"
date:   2015-08-05 15:14:00
description: Learn how to hide empty UITableView cells in your iOS apps.
# categories: blog geolocation geofencing latitude longitude
tags: ios UI UX UITableView programming development
---

Recently for a coding challenge I had to display a table full of search results from a web service. Always a fun challenge but when you really want to impress sometimes this:

![UITableView with empty cells]({{ site.baseurl }}/images/jeter-lines.png){: .img-responsive .center-block }

just looks a bit off putting if you're trying to show off your UI/UX skills. So how can we get rid of these empty cells? Well one way would be to add a UIView to your custom UITableViewCells, give it a background color and set it just at the bottom of your UITableViewCell. Then set the UITableViewCell separator property to None and call it a day.

I won't go into why that's such an absolutely awful idea, just so long as I'm able to hopefully convince you that it is and that there is a much easier and smarter way to get the results we want. So how exactly can we get rid of those cells?! For reference here's a diagram of what the UITableView component looks like with all the bells and whistles.

![UITableView UI components]({{ site.baseurl }}/images/uitableview-components.png){: .img-responsive .center-block }

We can see the table has two sections, each with two rows. We can also see the table's header view and the table's footer view. Most apps don't bother with the table header and footer views but what would happen should we create a new UIView and assign that view to the table's footer view? We can do that with the following code. Here we have the the IBOutlet for the table defined as `_resultsTable`

<pre><code class="objc">- (void)viewDidLoad {

    [super viewDidLoad];
    _resultsTable.tableFooterView = [UIView new];

    ...
</code></pre>

Setting the tableFooterView property to a new UIView takes care of this problem. The tableFooterView isn't the same as the footerViewForSection view. It's the footer view for the entire table. In setting it to an empty view, as we did in our viewDidLoad method from above, we get the following results:

![UITableView without empty cells]({{ site.baseurl }}/images/jeter-no-lines.png){: .img-responsive .center-block }

Setting an "empty" tableFooterView prompts iOS to show the tableFooterView instead of the empty cells in your table.

Hope this helps!
