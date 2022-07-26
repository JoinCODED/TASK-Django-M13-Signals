I haven't had my coffee yet :(

## Setup

1. Fork and clone [this repository](https://github.com/JoinCODED/TASK-Django-M13-Signals).
2. Install the requirements using `pip install -r requirements/dev.lock`.

## Signals Setup

1. Create a `signals.py` file inside of `coffeeshops`.
2. Go to `coffeeshops/apps.py` and add a `ready` method to `CoffeeshopsConfig` as follows:

   ```python
   class CoffeeshopsConfig(AppConfig):
       default_auto_field = "django.db.models.BigAutoField"
       name = "coffeeshops"

       def ready(self) -> None:
           import coffeeshops.signals
   ```

   - This will allow our signals to work when the app starts. You can read about this method [here](https://docs.djangoproject.com/en/4.0/ref/applications/#django.apps.AppConfig.ready).

## Emails

We want to be notified every time a new cafe owner joins our platform. To do that, we need to send an email each time a `CafeOwner` has been created.

1. Go to `coffeeshops/signals.py` and import `send_mail` from `django.core.mail` (we will need this function later).
2. Create a function called `send_new_owner_email`, which takes in a `sender`, `instance`, `created`, and `**kwargs`.
   - Use the `send_mail` function (read about it [here](https://docs.djangoproject.com/en/4.0/topics/email/#quick-example)) if `created` is `True`.
   - Send an email with the subject as `New Cafe Owner` and the body of the email as `A new cafe owner has joined named <FULL_NAME_OF_CAFE_OWNER>`. For the `from_email` you can use `"sender@test.com"` and `recipient_list` can be `["receiver@test.com"]`.
3. Decorate your `send_new_owner_email` with `register`, which is imported from `django.dispatch` (read about decorators [here](https://www.programiz.com/python-programming/decorator)).
4. Add `post_save` (imported from `django.db.models.signals`) to your `register` decorator as the first argument and set the `sender` parameter equal to `CafeOwner` (imported from `coffeeshops.models`).
   - Read about the `post_save` signal [here](https://docs.djangoproject.com/en/4.0/ref/signals/#django.db.models.signals.post_save).
5. Test out your signal by creating an admin account (using `./manage.py createsuperuser`), starting the server, logging in to the admin site, and creating a new `CafeOwner`.

   - You should see a new login `TASK-Django-M13-Signals/emails/TIMESTAMP.log` (where `timestamp` is hyphenated numbers) that should look like the following:

     ```text
     Content-Type: text/plain; charset="utf-8"
     MIME-Version: 1.0
     Content-Transfer-Encoding: 7bit
     Subject: New Cafe Owner
     From: sender@test.com
     To: receiver@test.com
     Date: Sat, 23 Jul 2022 22:26:35 -0000
     Message-ID: <165861519536.9692.1816094537173382016@YOUR_COMPUTER_USERNAME>

     A new cafe owner has joined named Guido van Rossum.
     -------------------------------------------------------------------------------
     ```

## Slugging

[Slug](https://docs.djangoproject.com/en/4.0/glossary/#term-slug) is a newspaper term and a short label for something. It contains only letters, numbers, underscores, or hyphens. They’re generally used in URLs. We want to auto-generate slugs for our `CoffeeShop` instances.

1. Create a function called `slugify_coffee_shop` in `coffeeshops/signals.py` and decorate it with a `pre_save` signal that has `CoffeeShop` as its sender (read about `pre_save` signals [here](https://docs.djangoproject.com/en/4.0/ref/signals/#pre-save)).
2. Check if `instance` does not have a `slug`.
3. If our `instance` does not have a slug then just set `instance.slug` equal to `create_slug(instance)` where `create_slug` is imported from `utils` (i.e., `from utils import create_slug`).
4. Test out your signal on the admin site, by creating a new `CoffeeShop`, you should see a `slug` assigned to your `CoffeeShop` after saving.

**BONUS:** if you can tell an instructor why a `pre_save` signal must be used for our `slugify_coffee_shop` receiver and not a `post_save` signal, you get extra points.

## Address Manager

We want to make sure that an address object always exists for a `CoffeeShop` even if one was never created or if it is deleted.

### Create a Default Address

1. Create a function called `add_default_address` in `coffeeshops/signals.py` and decorate it with a `post_save` signal that has `CoffeeShop` as its sender.
2. Check if the `instance` has been `created`, and if `instance.location` is empty.
   1. Create a `CoffeeShopAddress` object.
   2. Set `instance.location` equal to the new `CoffeeShopAddress` object you've created.
   3. Save your instance.
3. Test out your signal on the admin site by creating a new `CoffeeShop` and leaving the `location` field empty, you should see a default `location` assigned to your `CoffeeShop` after saving.

### Restore a Default Address

1. Create a function called `restore_default_address` and decorate it with a `post_delete` signal that has `CoffeeShopAddress` as its sender (read about `post_delete` signals [here](https://docs.djangoproject.com/en/4.0/ref/signals/#post-delete)).
2. Get the coffee shop from the deleted `instance` (hint: use the related name `coffee_shop`) and store it in a variable called `coffee_shop`.
3. Create a new `CoffeeShopAddress` instance.
4. Set `coffee_shop.location` equal to the new `CoffeeShopAddress` object you've created.
5. Save your `coffee_shop` instance.
6. Test out your signal on the admin site, by deleting a location from any `CoffeeShop`, you should see a default `location` assigned to the `CoffeeShop`.

**BONUS:** if you can tell an instructor why a `post_delete` signal must be used for our `restore_default_address` receiver and not a `pre_delete` signal, you get extra points.

## Inventory Management

We want to manage our `Drink` inventory and mark `is_out_of_stock` if our drinks go out of stock.

1. Create a function called `slugify_coffee_shop` in `coffeeshops/signals.py` and decorate it with a `pre_save` signal that has `Drink` as its sender.
2. Set `instance.is_out_of_stock` to `False` if `instance.stock_count` is greater than zero and vice-versa.
3. Test out your signal on the admin site, by setting a stock count greater than zero to a drink, you should see `is_out_of_stock` as `False` after saving.
