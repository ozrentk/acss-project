# Implementing the project task

Remember to add `debugger` directive where needed.
You can debug the code in Dev Tools.

Note that this tutorial does NOT include Redux or Zustand.

## Creating application and templating

1. Install libs
  - React: **react** and **react-dom**
  - Router: **react-router** and **react-router-dom**
  - Bootstrap to make your life easier: **react-bootstrap**

2. Create templates and link them together 
  - see public/template_xyz.html, e.g. public/template_customers.html
  - you will 'slice' these templates to components
  - your app is here: http://localhost:3000/
  - your templates are available using Live Server, e.g.: http://127.0.0.1:5791/public/template_home.html
    (VS Code uses arbitrary port)

## Basic application setup

3. Take templates and create components out of them (slice templates to components)
  - copy html code to simple function components
  - e.g. src/components/Customers.js with search, table, row and pager components
  - also extract navigation to separate component like src/components/Navigation.js
  - use these components in e.g. App.js
  - at the and you should have a working Customers page that is static
  - note: you will see a LOT of "Warning: Invalid DOM property `class`. Did you mean `className`?"
    That just means you have a native HTML that you need to convert to React-style HTML (class to className)

4. Set up routing
  - import router components where you need them, usually in App component
    (BrowserRouter as Router, Routes, Route)
  - create routing structure using router components and link routes to components

5. Set up navigation
  - replace <a> witk <Link> inside Navigation.js
  - ...instead of: <a class="nav-link" href="template_customers.html">Customers</a>
  - ...do this: <Link class="nav-link" to="/customers">Customers</Link>
  - ...also for login, register, profile...
  - observe in Dev Tools that there are no server requests

6. Proceed to implement other components that should be available on routes
  - home page, login, profile, register
  - now you should have a static React website with routing

## Implementing customers / acsounts / items search and display

7. Implement modal (CustomersTable)
  - use React bootstrap integration for this
  - see: https://react-bootstrap.netlify.app/docs/components/modal/#static-backdrop
    - see: Modal size, set it to "xl"
    - see: static backdrop, get rid of it
    - remove footer
  - check if opening modal using button works

8. Connect modal to table row (CustomersRow, CustomersTable)
  - now proceed to implement opening modal using event-prop
  - see: https://react.dev/learn/responding-to-events#passing-event-handlers-as-props
  - wire up dialog-show state to the row click
  - set up the proper content for the dialog (the one from the template)
  - you can now see that table inside modal is ready to slice it in smaller pieces, so do it
    (AccountsTable, AccountsRow)
  - ditch the "Launch static..." button

9. Integrate Account component
  - when you click link in AccountsRow table, you should navigate to Account component
  - create route to it in `App.js`
  - Use `<Link>` instead of `<a>` in 
  - Account component is probably empty one, there should be table with account items inside as in `template_account_items.html` - implement it (you also need AccountItem or AccountRow for rows)
  - Instead of link, attach event to the table row and navigate to the same path 
  - See: https://reactrouter.com/en/main/hooks/use-navigate

10. Ensure back-end connectivity
  - start back-end server
    Note: you can run it without authentication, like `npm run start`
  - start front-end
  - if both are working on the same port, you can change e.g. front-end to work on 3001
    - add .env file to front-end app root, and add line `PORT=3001` to move front-end app to port 3001
  - implement `useEffect()` hook to fetch data from json server when `Customers` component loads
    ```
    useEffect(() => {
      const fetchData = async () => {
        const response = await fetch(`http://localhost:3000/customer`);
        const data = await response.json();
        console.log(data);
      }
      fetchData();
    }, []);
    ```
  - ensure that Dev Tools show data received from server

11. Use data from server in your app (CustomersTable)
  - you need state for the customers (`useState()` hook), which will be an array
    ```
    const [customers, setCustomers] = useState([]);
    ```
  - when customers are loaded, use setCustomers to set that state
    ```
    useState(data);
    ```
  - in returned HTML markup, use `map()` to return HTML for each customer
    ```
    {customers.map((customer) => <CustomersRow key={customer.id} customer={customer} rowClicked={rowClicked} />)}
    ```
  - accept the passed prop `customer` in your function component and include it in row
    ```
    function CustomersRow({ customer, rowClicked }) {
      return (
        <tr onClick={rowClicked}>
          <td>{customer.id}</td>
          <td>{customer.name}</td>
          <td>{customer.surname}</td>
          <td>{customer.email}</td>
          <td>{customer.telephone}</td>
          <td>{customer.cityId}</td>
        </tr>
      );
    }
    ```
12. Introduce filter (CustomersSearch, CustomersTable, Customers)
  - for filter, you need a state where there is a search filter (CustomersSearch)
    (filter, setFilter)
  - you will also need ref for the input, and event handler for when input changes
  - final JS code:
    ```
    const [filter, setFilter] = useState("");
    const filterRef = useRef();

    const onSearchChange = (e) => {
      setFilter(e.target.value);
    }
    ```
  - final input markup
    ```
    <input type="text" ref={filterRef} class="form-control" placeholder="Username" onChange={onSearchChange} />
    ```
  - filter state is in `CustomersSearch` and dataset state is `CustomersTable`, let's lift both to `Customers` component

13. Filter state to implement filtering (CustomersSearch, CustomersTable, Customers)
  - state should be in `Customers`:
    ```
    const [customers, setCustomers] = useState([]);
    const [filter, setFilter] = useState("");
    ```
  - also, move `fetch()` to where state is; it's not mandatory but it's easier that way
  - create event handlers for setting customers and filter and pass them to components that need them
    ```
    useEffect(() => {
      const fetchData = async () => {
        const response = await fetch(`http://localhost:3000/customer?q=${filter}`);
        const data = await response.json();
        //console.log(data);
        setCustomers(customers);
      }
      fetchData();
    }, [filter]);

    const onSetFilter = (filter) => {
      setFilter(filter);
    };

    ...

    <CustomersSearch onSetFilter={onSetFilter} />
    <CustomersTable customers={customers} />
    ```
  - that way you:
    - set `customers` state when `fetch()` is done
    - pass `customers` state to `CustomersTable` for display
    - set `filter` state in `CustomersSearch` via event-prop
    - use `filter` state for `fetch()` inside `useEffect()`
    - NOTE: `useEffect()` runs only when filter is affected
  - now the dataset is filtered according to what is inside search box

14. Implement debouncing (Customers)
  - we don't want to immediatelly filter, so we debounce using setTimeout()
  - you need to:
    - add timeout identifier variable before `useEffect()`
      ```
      let fetchCustomersTimeout = null;
      ```
    - return cleanup function from `useEffect()`
      ```
      return () => {
        clearTimeout(fetchCustomersTimeout);
      }
      ```
    - use setTimeout() to run `fetch()`
      ```
      const fetchData = async () => {
        const response = await fetch(`http://localhost:3000/customer?q=${filter}`);
        const data = await response.json();
        //console.log(data);
        setCustomers(data);
      }

      let fetchCustomersTimeout = setTimeout(fetchData, 500);
      ```
  - now the dataset is not immediately filtered on each keypress, but delayed for 500ms

15. Implement paging (Customers, CustomersPager)
  - we need to fetch just 10 by 10 items
  - for API EP's see: https://github.com/typicode/json-server
  - while calling `fetch()`, add `_page=1` and `_limit=10` as query string parameters, and you should get first 10 filtered items
    ```
    const response = await fetch(`http://localhost:3000/customer?q=${filter}&_page=1&_limit=10`);
    ```
  - page should be state, so:
    - add a state for that (initially should be 1)
      ```
      const [page, setPage] = useState(1);
      ```   
    - use it in previous fetch
      ```
      const response = await fetch(`http://localhost:3000/customer?q=${filter}&_page=${page}&_limit=10`);
      ```
    - set it on button click in `CustomersPager` (use event-prop to pass event from `CustomersPager`)
      - `Customers`
      ```
      const onSetPage = (page) => {
        setPage(page);
      };

      ...

      <CustomersPager onSetPage={onSetPage} />
      ```
      - `CustomersPager`
      ```
      const onSetPage = () => {
        console.log("onSetPage");
      };

      ...

      <CustomersPager onSetPage={onSetPage} />
      ```
  - inside CustomersPager we have anchors, and to simplify things we will render them as buttons
  - also we will add page number values to buttons
    ```
    <nav>
      <ul class="pagination">
        <li class="page-item"><button type="button" class="page-link" value="1" onClick={props.onSetPage}>1</button></li>
        <li class="page-item"><button type="button" class="page-link" value="2" onClick={props.onSetPage}>2</button></li>
        <li class="page-item"><button type="button" class="page-link" value="3" onClick={props.onSetPage}>3</button></li>
      </ul>
    </nav>
    ```
  - and now we fix `onSetpage()` handler in `Customers`
    ```
    const onSetPage = (e) => {
      console.log("onSetPage", e.target.value);
      setPage(e.target.value);
    };
    ```
  - ensure that paging and filtering work now

16. Fetch data related to customer (CustomersTable)
  - when customer is clicked, we have to run `fetch()` with that customer as a parameter of fetch, and feed some new state where related records will be stored
  - pay attention that selected customer id is then also a state
  - of course, we need also to use that state for rendering 
    ```
    const [customerId, setCustomerId] = useState(null);
    const [accounts, setAccounts] = useState([]);
    ```
  - add `fetch()` to `CustomersTable`, handler `rowClicked()`
    ```
    const fetchData = async () => {
      const response = await fetch(`http://localhost:3000/bill?customerId=${customerId}`);
      const data = await response.json();
      //console.log(data);
      setAccounts(data);
    }
    fetchData();
    ```
  - pass `accounts` to `AccountsTable` and render them (see how it works in CustomersTable)
    ```
    ...
    <tbody>
      {accounts.map((account) => <AccountsRow key={account.id} account={account} />)}
    </tbody>
    ...
    ```

17. Fetch data related to credit cards (CustomersTable)
  - we now have just a part of the data (id, billNo, date), second part should come from credit card data; solution is to query credit card data like this: `http://localhost:3000/creditCard?id=1290&id=2850&id=2631`
  - after we get both data sets, we can merge them using mapping function
    ```
    const fetchData = async () => {
      // Get bills
      const response = await fetch(`http://localhost:3000/bill?customerId=${customer.id}`);
      const data = await response.json();
      console.log(data);

      // Get credit cards
      const ccIds = data.filter(bill => bill.creditCardId).map((bill) => bill.creditCardId);
      const ccQuery = ccIds.map((cc) => `id=${cc}`).join("&");
      // Example: http://localhost:3000/creditCard?id=1290&id=2850&id=2631
      const ccResponse = await fetch(`http://localhost:3000/creditCard?${ccQuery}`);
      const ccData = await ccResponse.json();
      console.log(ccData);

      // Join bill and credit card data
      const joinedData = data.map((bill) => {
        const foundCc = ccData.find((cc) => cc.id == bill.creditCardId);
        const item = {...foundCc, ...bill};
        item.expiration = item.expirationMonth ? `${item.expirationMonth}/${item.expirationYear}` : "";
        return item;
      })
      console.log(joinedData);

      setAccounts(joinedData);
    }
    fetchData();
    ```

18. Fetch account item data
  - in `AccountsRow` you can use `useNavigate()` to programatically navigate to `/account` URL
  - on row click you should also pass parameter for account id
    ```
    const accountsRowClick = (e) => {
      //console.log(account.id);
      navigate(`/account/${account.id}`);
    };
    ```
  - for this you need child route
    ```
    <Route path="/account" element={<Account />}>
      <Route path=":accountId" element={<Account />} />
    </Route>
    ```
  - in `Account` you can use `useparams()` to retrieve id from parameter
    ```
    import { useParams } from 'react-router-dom';
    ...
    let { accountId } = useParams();
    ```
  - now you can `fetch()` data
    ```
    const response = await fetch(`http://localhost:3000/item?billId=${accountId}`);
    ```
  - map account items correctly in return HTML markup, and implement `AccountItem` component

19. Fetch product data
  - what we have is another case where fetched data is just a part of whole data
  - we need also data on products, e.g. `http://localhost:3000/product?id=1&id=2&id=3`
  - use the same technique as before

20. Implement account item paging
  - same as for Customers component, but now on Account component

## Implementing security

21. Collect register form data (Register)
  - see: https://legacy.reactjs.org/docs/uncontrolled-components.html
    (for class components, but it's almost the same for function components)
  - create `useRef()` for all form inputs and ref them on these inputs
    ```
    const nameInputRef = useRef();
    ...
    const avatarFileInputRef = useRef();
    ...
    <label htmlFor="nameInput" class="form-label">Name</label>
    <input type="name" class="form-control" id="nameInput" ref={nameInputRef} />
    ...
    <label htmlFor="avatarFileInput" class="form-label">Avatar</label>
    <input type="file" accept="image/png, image/jpeg" class="form-control" id="avatarFileInput" ref={avatarFileInputRef} />
    ```
  - for form, handle `onSubmit` and inside that handler check if you get values of ref'ed controls
    ```
    e.preventDefault();
    console.log(
      nameInputRef.current.value,
      usernameInputRef.current.value,
      passwordInputRef.current.value,
      avatarFileInputRef.current.value
    );
    ```

22. Send register form data to the server (Register)
  - see: https://developer.mozilla.org/en-US/docs/Web/API/fetch
  - see: https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch
  - Note: here you **must** run server with authentication, like `npm run start-auth`, otherwise you won't have `login` and `register` endpoints
  - in `onFormSubmit()` handler, implement `fetch()` call with POST method to send the data to server
    ```
    const postData = async (data) => {
      const response = await fetch("http://localhost:3000/auth/register", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(data),
      });

      const _ = await response.json();
    }

    postData({
      "name": nameInputRef.current.value,
      "email": usernameInputRef.current.value,
      "password": passwordInputRef.current.value
    });
    ```
  - make sure that registering works (there should be new entry in `User.js` file on server)

23. Pull data on user from server after registering
  - imediatelly after registering, you can implicitly login user, and get token
  - we will naively use localstorage for the token
  - add another fetch after awaiting the last response
    ```
    delete data.name;
    const loginResponse = await fetch("http://localhost:3000/auth/login", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(data),
    });

    const loginResult = await loginResponse.json();
    localStorage.setItem('access_token', loginResult.access_token);
    
    // Test
    console.log(localStorage.getItem('access_token'));
    ```

24. Issue authenticated requests to server
  - when you try to get Customers, there is a runtime error
  - we can avoid this and send jwt token that is now in the localstorage to server,
    when sending requests
  - in Customers conponent, add `Authorization` header to `fetch()` call
    ```
    const access_token = localStorage.getItem('access_token');
    const options = {
      headers: {
        'Authorization': `Bearer ${access_token}`
      }
    };
    const response = 
      await fetch(
        `http://localhost:3000/customer?q=${filter}&_page=${pager}&_limit=10`,
        options);
    ```
  - ensure that server returns data now
  - find other `fetch` calls (not login/register) and authenticate them

25. Implement login page (Login)
  - use refs for username (maps to email) and password
  - fetch request to POST `http://localhost:3000/auth/login`
  - set localstorage as in `register`
  - at the end, navigate to `/customers`
  - see: https://reactrouter.com/en/main/components/navigate

## Handling form data

26. Show data on profile page (Profile, Login, Register)
  - to get profile data, one should have username or id available
    - change login procedure and store user id to localstore
    - run authorized `fetch()` for user, e.g. `http://localhost:3000/User?email=admin@email.com`
      ```
      const access_token = localStorage.getItem('access_token');
      const options = {
        headers: {
          'Authorization': `Bearer ${access_token}`
        }
      };
      const userResponse = 
        await fetch(
          `http://localhost:3000/User?email=${usernameInputRef.current.value}`,
          options);
      const userResult = await userResponse.json();
      localStorage.setItem('user_id', userResult[0].id);
      ```
    - do this both for login and register process (Login, Register)
  - `useEffect()` to run `fetch()` (Profile)
    - for example, `http://localhost:3000/user?id=${user_id}`
  - you need state
    ```
    const [user, setUser] = useState({
      name: "",
      email: "",
      password: ""
    });
    ```
  - when data is fetched, set the state
  - use the form to display the state
  - since you need to be able to change the state, you need onChange handler, which makes the component controllable
  - use this neat trick with 2nd order function to update state on such component
    ```
    <div class="mb-3 mx-5">
      <label htmlFor="nameInput" class="form-label">Name</label>
      <input type="text" class="form-control" id="nameInput" value={user.name} onChange={onProfileChanged('name')} />
    </div>
    <div class="mb-3 mx-5">
      <label htmlFor="usernameInput" class="form-label">Username</label>
      <input type="text" class="form-control" id="usernameInput" value={user.email} onChange={onProfileChanged('email')} />
    </div>
    
    ...

    // 2nd order function!
    const onProfileChanged = (attr) => {
      return (e) => {
        setUser({ ...user, [attr]: e.target.value });
      };
    }
    ```

27. Update users profile
  - you need to send data to server to store the profile
  - make sure that you send PUT request with body
    Example: PUT http://localhost:3000/user/1
  - make sure you construct proper options for the request
  - options example: PUT method, proper content type, JWT token, body

28. Validate users profile data
  - we will manually validate data here
  - procedure should be: 
    - before submitting the form manually check if the form data is correct
    - if correct, proceed to send data
      ```
      // Form validation
      const newErrors = { 
        name: !user.name ? "Name must not be empty" : "",
        email: !user.email ? "E-mail must not be empty" : "",
        password: !user.password ? "Password must not be empty" : "",
      };

      if(!newErrors.name && !newErrors.email && !newErrors.password) {
        postData({
          "name": user.name,
          "email": user.email,
          "password": user.password
        });
      }

      setValidated(true);
      setError(newErrors);
      ```
  - we're using bootstrap, so we play by the rules of it
    - see: https://getbootstrap.com/docs/5.2/forms/validation/
    - set `noValidate` on `form`
    - set `is-valid` or `is-invalid` class on control that is valid or invalid
    - add `<div>` with `valid-feedback` or `invalid-feedback` after the control, with text
      ```
      // Error helpers
      const errorClassFor = (name) => 
        !validated ? "" : !error[name] ? "is-valid" : "is-invalid";

      const errorFeedbackFor = (name) => {
        if(!validated)
          return "";
        else if(error[name])
          return <div class="invalid-feedback">{error[name]}</div>;
      }
      ...
      <div class="mb-3 mx-5">
        <label htmlFor="nameInput" class="form-label">Name</label>
        <input 
          type="text" 
          id="nameInput" 
          value={user.name} 
          class={"form-control " + errorClassFor('name')} 
          onChange={onProfileChanged('name')} />
        {errorFeedbackFor('name')}
      </div>
      ```

## Editing collections

29. Delete account item (AccountItem, Account)
  - attach button handler in AccountItem and pass it via event-prop
    ```
    <td><button type="button" class="btn btn-danger" onClick={() => accountItemDelete(item.id)}>Delete</button></td>
    ```
  - write delete handler in Account, using fetch
  - at the end, update state with removed account item
    ```
    const accountItemDelete = (id) => {
      const deleteData = async () => {
        const options = requestOptions();
        options.method = "DELETE";
        const response = await fetch(`http://localhost:3000/item/${id}`, options);
        const _ = await response.json();

        setAccountItems(accountItems.filter((item) => item.id != id));
      }
      deleteData();
    }
    ```

30. Add an account item (Account)
  - you need a state to remember if form is open or closed
  - you also need a state for the form
  - don't forget you need all the data for the dropdowns, and information if a dropdown is 
    selected or not
    ```
    const [newItemOpen, setNewItemOpen] = useState(false);
    const [categories, setCategories] = useState([]);
    const [selectedCategory, setSelectedCategory] = useState("");
    const [subcategories, setSubcategories] = useState([]);
    const [selectedSubcategory, setSelectedSubcategory] = useState("");
    const [products, setProducts] = useState([]);
    const [selectedProduct, setSelectedProduct] = useState("");
    const [selectedQuantity, setSelectedQuantity] = useState(1);

    ```
  - for markup, you need a button to open/close the form and the form itself
    ```
    const onNewItemClick = () => {
      setNewItemOpen(!newItemOpen);
    }
    ...
    <div class="row">
      <button type="button" class="btn btn-primary" onClick={onNewItemClick}>
        {!newItemOpen ? "Add" : "Cancel add"}
      </button>
      {newItemOpen && 
        <h2>New item form here</h2>
      }
    </div>
    <hr />
    ```
  - implement a form (markup instead of that `<h2>`)
    ```
    const onFormSubmit = (e) => {
      e.preventDefault();
    }

    ...

    <form onSubmit={onFormSubmit}>
      <div class="mb-3 mx-5">
        <label htmlFor="categorySelect" class="form-label">Category</label>
        <select 
          id="categorySelect" 
          class="form-select" 
          value={selectedCategory}
          onChange={(e) => setSelectedCategory(e.target.value)}>
          <option value="">--Please select--</option>
          {categories.map((cat) => 
            <option key={cat.id} value={cat.id}>{cat.name}</option>)}
        </select>
      </div>
      ...
      <div class="mb-3 mx-5">
        <label htmlFor="quantitySelect" class="form-label">Quantity</label>
        <select
          id="quantitySelect" 
          class="form-select"
          value={selectedQuantity}
          onChange={(e) => setSelectedQuantity(e.target.value)}>
          <option value="1">1</option>
          <option value="2">2</option>
          <option value="3">3</option>
        </select>
      </div>
      <button type="submit" class="btn btn-primary mx-5 mt-3">Confirm add</button>
    </form>
    ```
  - we have 4 dropdowns, of which 3 will be loaded from data from server
    - first one will be loaded when page loads (useEffect())
      ```
      useEffect(() => {
        //console.log("fetching categories");
        const fetchData = async () => {
          const options = requestOptions();
          const response = await fetch(
            `http://localhost:3000/category`,
            options);
          const data = await response.json();

          setCategories(data);
        }
        fetchData();
      }, []);
      ```
    - second one will be loaded when first one is selected (useEffect() with dependency)
      ```
      useEffect(() => {
        //console.log(`fetching subcategories for category ${selectedCategory}`);
        const fetchData = async () => {
          const options = requestOptions();
          const response = await fetch(
            `http://localhost:3000/subcategory?categoryId=${selectedCategory}`,
            options);
          const data = await response.json();

          setSubcategories(data);
        }
        fetchData();
      }, [selectedCategory]);
      ```
    - third one will be loaded when second one is selected (useEffect() with dependency)
      ```
      useEffect(() => {
        //console.log(`fetching products for subcategory ${selectedSubcategory}`);
        const fetchData = async () => {
          const options = requestOptions();
          const response = await fetch(
            `http://localhost:3000/product?subCategoryId=${selectedSubcategory}`,
            options);
          const data = await response.json();

          setProducts(data);
        }
        fetchData();
      }, [selectedSubcategory]);
      ```
    - additionally, reset dropdowns when their "ancestor" dropdown gets changed
      ```
      setCategories(data);

      // When categories are fetched, reset selected category and all other data
      setSelectedCategory("");
      setSubcategories([]);
      setSelectedSubcategory("");
      setProducts([]);
      setSelectedProduct("");
      setSelectedQuantity(1);
      ```
      ```
      setSubcategories(data);

      // When subcategories are fetched, reset selected subcategory and products
      setSelectedSubcategory("");
      setProducts([]);
      setSelectedProduct("");
      setSelectedQuantity(1);
      ```
      ```
      setProducts(data);

      // When products are fetched, reset selected product
      setSelectedProduct("");
      setSelectedQuantity(1);
      ```
  - when you add a product, send data to the server and add that product to state
    ```
    const onFormSubmit = (e) => {
      e.preventDefault();

      const postData = async (data) => {
        const options = requestOptions();
        options.method = "POST";
        options.headers["Content-Type"] = "application/json";
        options.body = JSON.stringify(data);
        const response = await fetch("http://localhost:3000/item", options);
    
        const _ = await response.json();
      }

      const newAccountItem = {
        "guid": null,
        "billId": accountId,
        "quantity": selectedQuantity,
        "productId": selectedProduct,
        "totalPrice": 123
      };
      //console.log(newAccountItem);

      postData(newAccountItem);
    }
    ```

## Exercises

31. Finish paging functionality (number of pages)
32. Fetch also city information on Customers component
33. Implement paging on Accounts component
34. On account items, show customer data, account data and totals
35. Validation on Profile component
36. Error handling
37. Modal confirmation dialog when deleting item
38. Toast when creating/deleting item
39. Validation when adding account item
40. Generate GUID and calculate correct total price when adding account item

