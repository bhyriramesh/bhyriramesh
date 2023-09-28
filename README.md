
import pytest
from selenium import webdriver

@pytest.fixture
def driver():
    driver = webdriver.Chrome(executable_path="../drivers/chromedriver.exe")
    yield driver
    driver.quit()

@pytest.mark.parametrize("username, password", [("standard_user", "secret_sauce"), ("performance_glitch_user", "secret_sauce")])
def test_successful_login_and_logout(driver, username, password):
    driver.get("https://www.saucedemo.com")
    driver.find_element_by_id("user-name").send_keys(username)
    driver.find_element_by_id("password").send_keys(password)
    driver.find_element_by_id("login-button").click()
    
    assert "inventory" in driver.current_url

    # Perform Logout
    driver.find_element_by_id("react-burger-menu-btn").click()
    driver.find_element_by_id("logout_sidebar_link").click()
    
    assert "inventory" not in driver.current_url

def test_locked_out_user_login_failure(driver):
    driver.get("https://www.saucedemo.com")
    driver.find_element_by_id("user-name").send_keys("locked_out_user")
    driver.find_element_by_id("password").send_keys("secret_sauce")
    driver.find_element_by_id("login-button").click()
    
    error_message = driver.find_element_by_css_selector("h3").text
    assert "Epic sadface: Sorry, this user has been locked out." in error_message
test_cart.py:

python
Copy code
import pytest
import random
from selenium import webdriver

@pytest.fixture
def driver():
    driver = webdriver.Chrome(executable_path="../drivers/chromedriver.exe")
    yield driver
    driver.quit()

def add_random_items_to_cart(driver, count):
    for _ in range(count):
        items = driver.find_elements_by_css_selector(".inventory_item")
        random_item = random.choice(items)
        item_name = random_item.find_element_by_css_selector(".inventory_item_name").text
        random_item.find_element_by_css_selector(".btn_primary").click()
        assert f"REMOVE {item_name}" in driver.page_source

def test_retained_items_in_cart_after_logout(driver):
    driver.get("https://www.saucedemo.com")
    driver.find_element_by_id("user-name").send_keys("standard_user")
    driver.find_element_by_id("password").send_keys("secret_sauce")
    driver.find_element_by_id("login-button").click()
    
    add_random_items_to_cart(driver, 3)

    # Logout and login again
    driver.find_element_by_id("react-burger-menu-btn").click()
    driver.find_element_by_id("logout_sidebar_link").click()
    assert "inventory" not in driver.current_url

    driver.find_element_by_id("user-name").send_keys("standard_user")
    driver.find_element_by_id("password").send_keys("secret_sauce")
    driver.find_element_by_id("login-button").click()

    # Verify items are retained in the cart
    driver.find_element_by_css_selector(".shopping_cart_container").click()
    assert "Your Cart" in driver.page_source

def test_place_order(driver):
    driver.get("https://www.saucedemo.com")
    driver.find_element_by_id("user-name").send_keys("standard_user")
    driver.find_element_by_id("password").send_keys("secret_sauce")
    driver.find_element_by_id("login-button").click()
    
    # Add the highest and lowest priced items to cart
    add_random_items_to_cart(driver, 2)

    # Navigate to checkout information page and submit the form
    driver.find_element_by_css_selector(".shopping_cart_container").click()
    driver.find_element_by_css_selector(".checkout_button").click()
    driver.find_element_by_id("first-name").send_keys("John")
    driver.find_element_by_id("last-name").send_keys("Doe")
    driver.find_element_by_id("postal-code").send_keys("12345")
    driver.find_element_by_css_selector(".cart_button").click()

    # Verify order summary
    assert "Checkout: Overview" in driver.page_source

    # Place the order
    driver.find_element_by_css_selector(".btn_action.cart_button").click()
    
    # Verify order confirmation
    assert "THANK YOU FOR YOUR ORDER" in driver.page_source
