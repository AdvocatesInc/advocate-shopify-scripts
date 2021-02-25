# Advocate Shopify Conversion Tracking

1.  Insert the First script into the theme.

    1. In the store dashboard, click "Online Store" -> "Themes"

    2. Click the "Customize" button for the current theme

    3. At the bottom of the sidebar, click "Theme Actions" -> "Edit Code"

    4. In the code editor, click "Layout" -> "theme.liquid"

    5. In the `<head>` tag somewhere, paste:

    ```html
    <script>
     function clearCookie (name) {
        document.cookie = name + '=; expires=Thu, 01 Jan 1970 00:00:00 GMT'
      }

      function setCookie (name, value) {
        document.cookie = name + '=' + value;
      }

      function getSource () {
        const queryParams = new URLSearchParams(window.location.search)
        return queryParams.get('advref')
      }

      document.addEventListener('DOMContentLoaded', function() {
        var source = getSource();

        if (source) {
          setCookie('advref', source);
        }
      });
    </script>
    ```

2. Add the second script to the Checkout admin

    1. In the store dashboard, click "Settings" -> "Checkout"

    2. Scroll down to the "Order Processing" section

    3. In the "Additional Scripts" field, paste:

    ```html
    {% if first_time_accessed %}
      <script>
        function postJSON (url, json) {
          return fetch(url, {
            method: 'post',
            headers: {
              Accept: 'application/json, text/plain, */*',
              'Content-Type': 'application/json',
            },
            body: JSON.stringify(json),
          });
        }

        function getCookie (name) {
          var match = document.cookie.match(new RegExp('(^| )' + name + '=([^;]+)'));
          if (match) {
            return match[2];
          }
        }

        document.addEventListener('DOMContentLoaded', function() {
          var advCookie = getCookie('advref');

          if (advCookie) {
            var data = {
              order_number: '{{ order_number }}',
              customer: {
                id: '{{ customer.id }}',
                city: '{{ customer.default_address.city }}',
                state: '{{ customer.default_address.province }}',
                country: '{{ customer.default_address.country }}',
              },
              'purchased': [
                {% for line_item in checkout.line_items %}
                  {
                    product_id: '{{ line_item.product_id }}',
                    quantity: '{{ line_item.quantity }}',
                    title: '{{ line_item.product.title }}',
                    variant: '{{ line_item.product.variant }}',
                  },
                {% endfor %}
              ],
              currency: '{{ checkout.currency.iso_code }}',
              total: '{{ checkout.total_price | money }}',
            };

            postJSON('https://api.adv.gg/v1/register-conversion/', {
              source: advCookie,
              data: data,
            });
          }
        }, false);
      </script>
    {% endif %}
    ```
