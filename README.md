# Product-Inventory

## Introduction
During my last month at the Tech Academy I had the opportunity to work on developing a Hobby Inventory application using Python. This gave me experience working with other developers, fixing bugs, and using CRUD. On the [back end](#back-end-stories) I created a form that allowed people to input products, a table that allows users to see the details of their entries, as well as edit and delete them, and I was able to data scrape using  Beautiful Soup to give the user a product catalog and link them to a website where they could buy the product they saw. On the [front end](#front-end-stories) I adjusted the look of my app to compliment the standard layout and create a friendlier UI. Working on this project with a development team gave me a better understanding of how a project team operates in the tech world and improve my problem solving skills. Below are code snippets and screenshots from my project. 
I have included descriptions, code snippets, and screenshots from my project below!

## Back End Stories
* [Product Table](#product-table)
* [Beautiful Soup Product Catalog](#beautiful-soup-product-catalog)

### Product Table
One of my main tasks required me to develop a table that allowed the user to enter products they used, review the details, and update their entries.


    def index(request):
        get_products = Product.Products.all()      #Gets all the current hair products from the database
        context = {'products': get_products}      #Creates a dictionary object of all the products for the template
        return render(request, 'CurlJournal/curl_index.html', context)

    #View function to add a new product to the database
    def add_product(request):
        form = ProductForm(request.POST or None)     #Gets the posted form, if one exists
        if form.is_valid():                         #Checks the form for errors, to make sure it's filled in
            form.save()                             #Saves the valid form/hair product to the database
            return redirect('listHairProducts')                #Redirects to the index page, which is named 'curls' in the urls
        else:
            print(form.errors)                      #Prints any errors for the posted form to the terminal
            form = ProductForm()                     #Creates a new blank form
        return render(request, 'CurlJournal/curl_create.html', {'form':form})

    #View function to look up the details of a product
    def details_product(request, pk):
        pk = int(pk)                                #Casts value of pk to an int so it's in the proper form
        product = get_object_or_404(Product, pk=pk)   #Gets single instance of the hair product from the database
        context={'product':product}                  #Creates dictionary object to pass the product object
        return render(request,'CurlJournal/curl_details.html', context)

    #function to edit project in collection
    def edit_product(request, pk):
        pk = int(pk)
        product = get_object_or_404(Product, pk=pk)
        form = ProductForm(data=request.POST or None, instance=product)
        if request.method == 'POST':
            if form.is_valid():
                form2 = form.save(commit=False)
                form2.save()
                return redirect('listHairProducts')
            else:
                print(form.errors)
        return render(request, "CurlJournal/curl_edit.html", {'form': form})

    #function to delete product from collection
    def delete_product(request, pk):
        pk = int(pk)
        product = get_object_or_404(Product, pk=pk) #grabs specific product to be deleted
        if request.method == 'POST':
            product.delete()
            return redirect('listHairProducts')
        context = {'product': product,} #
        return render(request, "CurlJournal/curl_delete.html", context)
       
### Beautiful Soup Product Catalog
I wanted to create a way for the user's to easily access a catalog they could buy products from. I was able to data scrape from a large hair care website and display the product photo and description with a link to the products web page.

    def product_photos(request):
        source = requests.get('https://shop.naturallycurly.com/top-natural-hair-products') #Get website as an html document
        print(source.status_code) #used to debug to ensure code is successful
        soup = BS(source.content, 'html.parser') #Initial processing of the html by beautiful soup, soup is now a navigatable object
        nodes = soup.find_all(class_="product-item-details") #search for class name in HTML
        products = [] #creates an array to add the products to
        for node in nodes: #iterates through all the objects with class of node-title
            title = node.find('a').get_text() #Creates title for text tag
            link = node.find('a').get('href') #Sets link for equal to href tag
            image = node.find_previous(class_='show-for-sr').get('src') #grabs image for display
            url = link #Modifies link to full url
            product={'image': image, 'title': title, 'url': url} #creates article object dictionary with certain elements
            products.append(product) #adds item to the array
        context={'products':products} #creates dictionary for the products to pass through templates
        return render(request,'CurlJournal/curl_products.html', context)
        
*Jump to [Front End Stories](#front-end-stories), [Back End Stories](#back-end-stories), [Page Top](#product-inventory)*

## Front End Stories
Initially once the product table was created the "details" button was part of the table. In order to allow the user to know they could view the details of one entry I gave the "details" button a different design.The standard backround for all the apps made the text hard to read so I added a a solid background to create a nicer UX.

* [Product Index w/ Details button](#product-index)
* [Details Page w/ adjusted background](#details-page)

### Product Index

    <section >
        <div class="flex-container" id="productCollectionPage">
            <table class="table-striped">
                <tr>
                    <th class="col-md">Name</th>
                    <th class="col-md">Brand</th>
                    <th class="col-md">Purpose</th>
                    <th class="col-md">Results</th>
                    <th class="col-md">Use again?</th>
                </tr>
                {% for product in products %}     <!-- creates a new row for each product in the collection -->
                    <tr>
                        <td class="col-md">{{product.name}}</td>
                        <td class="col-md">{{product.brand}}</td>
                        <td class="col-md">{{product.purpose}}</td>
                        <td class="col-md">{{product.main_result}}</td>
                        <td class="col-md">
                            {% if product.repeat_use %} Yes {%else%} No {% endif %}
                        </td>

                        <td class="col-md-details"><a href="{{product.pk}}/Details"><button class="details-btn">Details</button></a></td>    <!-- creates a link for that specific product -->
                    </tr>
                {% endfor %}
            </table>
            <button class="primary-bright-button" type="button" onclick=" location.href='{% url 'createHairProduct' %}'">Add to Collection</button>
        </div>
        </div>
    </section>
    
    
### Details Page

    <section>
        <div class="flex-container footy" id="productDetails">
            <table>
                <tr>
                    <th class="category">Name:</th>
                    <td class="results">{{product.name}}</td>
                </tr>
                <tr>
                    <th class="category">Brand:</th>
                    <td class="results">{{product.brand}}</td>
                </tr>
                <tr>
                    <th class="category">Purpose:</th>
                    <td class="results">{{product.purpose}}</td>
                </tr>
                <tr>
                    <th class="category">Results:</th>
                    <td class="results">{{product.main_result}}</td>
                </tr>
                <tr>
                    <th class="category">Use Again:</th>
                    <td class="results"> {% if product.repeat_use %} Yes {% else %} No {% endif %} </td>
                </tr>
            </table>

            <hr />
            <div id="changeButtons">
            <button class="primary-bright-button" type="button" onclick=" location.href='{% url 'listHairProducts' %}'">Back to Collection</button>
            <a href="../Edit/"><button class="primary-bright-button" type="button">Edit Product Details</button></a>
            <a href="../Delete/"><button class="primary-bright-button" type="button">Delete Product</button></a>
            </div>
        </div>
    </section>
    
 *Jump to [Front End Stories](#front-end-stories), [Back End Stories](#back-end-stories), [Page Top](#product-inventory)*
