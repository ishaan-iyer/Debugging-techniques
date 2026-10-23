# Debug Log

**Explain how you used the the techniques covered (Trace Forward, Trace Backward, Divide & Conquer) to uncover the bugs in each exercise. Be specific!**

In your explanations, you may want to answer:

- What is the expected vs. actual output?
- If there is a stack trace, what useful information does it contain?
- Which technique did you use, on which line numbers?
- What assumptions did you have about each line of code, and which ones were proven to be wrong?

_Example: I noticed that the program should show pizza orders once a new order is made, and that it wasn't showing any. So, I used the trace forward technique starting on line 13. I discovered the bug on line 27 was caused by a typo of 'pzza' instead of 'pizza'._

_Then I noticed another bug ..._

## Exercise 1

I started with a trace forward from the order form because the homepage already loaded, but submitting an order did not produce a new pizza. Before the request could reach the route, running `python3 app.py` produced an `IndentationError: unexpected indent` at line 43. The traceback was useful because it identified the file and the exact line where Python stopped parsing; it showed that this was a syntax problem, not a database or template problem. Divide and conquer helped me ignore the unrelated homepage code and isolate the enum/database setup block around lines 25-59. That block contains several invalid fragments, including the over-indented line 43 and non-Python statements, so the application cannot start until they are removed.
 The second leftover loop at lines 43-45 reads `right_side[i]` instead of `right_side[j]`, which raises an `IndexError` when the right half still has an element. My assumption that each merge step preserved sorted order was disproved by these three lines. The expected list is `[1, 2, 3, 4, 5, 6, 7, 9]`; the actual program stops with an `IndexError` before printing a sorted list.
After isolating the parser errors, I traced the form values forward from `order.html` to `pizza_order_submit`. The form sends `order_name` and `pizza_size` (lines 8 and 26 of the template), but the route reads `name` and `size` (lines 141-142). Those lookups therefore return `None`, even though the form appears to submit correctly. I also found that the route ends at line 144; the pizza creation code was placed in a separate `pizza_bake_oven` function at line 147, where it is not called and contains syntax errors at lines 148-149. The code also uses `topping=` at line 157 even though the model field is named `topping_type` at line 71. Finally, the pizza is added to the session at line 159 but never committed, and `url_for('/')` at line 162 uses a URL instead of the endpoint name. My original assumption that the POST route would create and save the order was therefore wrong. The expected result is a committed pizza visible on the homepage; the actual result is that the app first fails to start, and after syntax errors are removed the submitted values and persistence path are still broken.

## Exercise 2

I used trace forward from the working homepage and clicked Submit. The first check, `python3 app.py`, produced an `IndentationError` at line 47 in the `results` route. The traceback narrowed the failure to the malformed `@app.json(...)` block and the non-Python statements on lines 47-52, before any HTTP request or API call can run. I used divide and conquer to separate this parser failure from the weather-data path: first the route must parse, then the request parameters and response mapping can be checked.

Tracing the submitted values forward exposed a form/route mismatch. The form sends `city` and `units` in `home.html` (lines 15 and 25), but the route reads `users_city` and `requested_units` at lines 54-55. Thus `city` and `units` become `None`. The API parameter at line 60 is named `place`, while OpenWeather expects `q` for the city. In the response mapping, line 69 asks for `result_json['main']['temperature']`, but the API field is `temp`; line 71 asks for an `education` field that weather responses do not provide. Later, line 83 contains the typo `spped` instead of `speed`. These errors explain why the expected Results page should show the selected city, temperature, humidity, description, and wind speed, while the actual behavior is either a startup syntax error or, after that is fixed, an invalid request/`KeyError` response mapping. The `get_letter_for_units` function also falls through to `K` for any value other than `imperial` or `metric`, so the requested `kelvin` option is intentionally supported by that fallback but should be verified after the input-name bug is fixed.

## Exercise 3

I ran `python3 main.py` and traced forward from `merge_sort(list_of_nums)` on line 9. Divide and conquer was especially useful here because merge sort recursively splits the list, so I inspected one small pair at a time. The merge comparison at line 24 uses `>` and copies the larger value first; for ascending output it should copy the smaller value first. The leftover-element loop at lines 39-41 never increments `k`, so later values overwrite the same position. The second leftover loop at lines 43-45 reads `right_side[i]` instead of `right_side[j]`, which can select the wrong element or raise an `IndexError`. My assumption that each merge step preserved sorted order was disproved by these three lines. The expected list is `[1, 2, 3, 4, 5, 6, 7, 9]`; the actual list is not correctly sorted.

I then traced backward from the required output `Index of 5`. `binary_search` is supposed to receive a sorted list, so I first treated the bad sort as one independent fault and inspected the search loop separately. At line 57, `(high + low) / 2` produces a float, but list indexes must be integers, so execution raises a `TypeError` when it reaches `arr[mid]`. Even after changing that to integer division, the loop at line 54 should use `low <= high`; with `low == high`, the final candidate is currently skipped. The function should return the integer midpoint when the value matches and `-1` only after all candidates have been eliminated. The expected result after fixing both functions is the sorted list above and index `4` for the value `5`; the actual program either fails while merging or fails in binary search before producing that result.`
