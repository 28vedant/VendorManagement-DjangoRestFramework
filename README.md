
# Vendors Management System Using Django

### Features:

1.Vendor Management: Create, retrieve, update, and delete vendor information.

2.Purchase Order Management: Create, retrieve, update, and delete purchase orders.

3.Vendor Performance Metrics: Retrieve performance metrics for vendors, including on-time delivery rate, quality rating average, average response time, and fulfillment rate.

4.Authentication and Authorization: Token-based authentication and permission checks ensure secure access to the API endpoints.


### Technologies Used

1.Django: Web framework for building the backend.

2.Django Rest Framework: Toolkit for building Web APIs.

3.Python: Programming language used for backend development.

4.SQLite: Database management system used for storing data
.
5.Token Authentication: Authentication mechanism for securing API endpoints.




## Deployment

To deploy this project run

```bash
  npm run deploy
```

Clone the repository:

```bash
git clone https://github.com/yourusername/vendor-management-api.git
```
Install dependencies:
```bash
pip install -r requirements.txt
```


Apply migrations:
```bash
python manage.py migrate
```
Run the development server:
```bash
python manage.py runserver
```
Access the API at http://localhost:8000/api/
## API Endpoints

### Vendors:

1.GET /api/vendors/: List all vendors.

2.POST /api/vendors/: Create a new vendor.

3.GET /api/vendors/<vendor_id>/: Retrieve a specific vendor.

4.PUT /api/vendors/<vendor_id>/: Update a specific vendor.

5.DELETE /api/vendors/<vendor_id>/: Delete a specific vendor.


### Purchase Orders:

1.GET /api/purchase_orders/: List all purchase orders.

2.POST /api/purchase_orders/: Create a new purchase order.

3.GET /api/purchase_orders/<order_id>/: Retrieve a specific purchase order.

4.PUT /api/purchase_orders/<order_id>/: Update a specific purchase order.

5.DELETE /api/purchase_orders/<order_id>/: Delete a specific purchase order.

### Vendor Performance:
1.GET /api/vendors/<vendor_id>/performance/: Retrieve performance metrics for a specific vendor.


### Data Models
#### 1. Vendor Model
```bash
# Fields:
# - name: CharField
# - contact_details: TextField
# - address: TextField
# - vendor_code: CharField
# - on_time_delivery_rate: FloatField
# - quality_rating_avg: FloatField
# - average_response_time: FloatField
# - fulfillment_rate: FloatField
```
#### 2. Purchase Order (PO) Model

```bash
# Fields:
# - po_number: CharField
# - vendor: ForeignKey
# - order_date: DateTimeField
# - delivery_date: DateTimeField
# - items: JSONField
# - quantity: IntegerField
# - status: CharField
# - quality_rating: FloatField
# - issue_date: DateTimeField
# - acknowledgment_date: DateTimeField
```
#### 3. Historical Performance Model

```bash # Fields:
# - vendor: ForeignKey
# - date: DateTimeField
# - on_time_delivery_rate: FloatField
# - quality_rating_avg: FloatField
# - average_response_time: FloatField
# - fulfillment_rate: FloatField
```
#### The backend logic includes calculations for various performance metrics like On-Time Delivery Rate, Quality Rating Average, Response Time, and Fulfilment Rate based on interactions recorded in the Purchase Order model.

## Views
### view for listing and creating objects.
```bash
class VendorListCreateView(generics.ListCreateAPIView):
    # authentication_classes = [TokenAuthentication]
    # permission_classes = [IsAuthenticated]
    queryset = Vendor.objects.all()
    serializer_class = VendorSerializer
```
###  view for retrieving, updating, and deleting objects.
```bash
class VendorRetrieveUpdateDeleteView(generics.RetrieveUpdateDestroyAPIView):
    # authentication_classes = [TokenAuthentication]
    # permission_classes = [IsAuthenticated]
    queryset = Vendor.objects.all()
    serializer_class = VendorSerializer
```

### view for listing and creating objects.but for purchase order
```bash 
class PurchaseOrderListCreateView(generics.ListCreateAPIView):
    # authentication_classes = [TokenAuthentication]
    # permission_classes = [IsAuthenticated]
    queryset = PurchaseOrder.objects.all()
    serializer_class = PurchaseOrderSerializer
```

###  view for retrieving, updating, and deleting objects
```bash
class PurchaseOrderRetrieveUpdateDeleteView(generics.RetrieveUpdateDestroyAPIView):
    # authentication_classes = [TokenAuthentication]
    # permission_classes = [IsAuthenticated]
    queryset = PurchaseOrder.objects.all()
    serializer_class = PurchaseOrderSerializer
```

###  view for retrieving a single object.(Performance:['on_time_delivery_rate', 'quality_rating_avg', 'average_response_time', 'fulfillment_rate'])
)
```bash
class VendorPerformanceView(generics.RetrieveAPIView):
    # authentication_classes = [TokenAuthentication]
    # permission_classes = [IsAuthenticated]
    queryset = Vendor.objects.all()
    serializer_class = VendorSerializer

    def retrieve(self, request, *args, **kwargs):
        instance = self.get_object()
        serializer = self.get_serializer(instance)
        return Response({'on_time_delivery_rate': serializer.data['on_time_delivery_rate'],
                 'quality_rating_avg': serializer.data['quality_rating_avg'],
                 'average_response_time': serializer.data['average_response_time'],
                 'fulfillment_rate': serializer.data['fulfillment_rate']})
        # return Response(serializer.data['on_time_delivery_rate', 'quality_rating_avg', 'average_response_time', 'fulfillment_rate'])
```

### View for Acknowledgement of Purchase Order
```bash
class AcknowledgePurchaseOrderView(generics.UpdateAPIView):
    # authentication_classes = [TokenAuthentication]
    # permission_classes = [IsAuthenticated]
    queryset = PurchaseOrder.objects.all()
    serializer_class = PurchaseOrderSerializer

    def create(self, request, *args, **kwargs):
        instance = self.get_object()
        instance.acknowledgment_date = request.data.get('acknowledgment_date')    #timezone.now()
        instance.save()
        response_times = PurchaseOrder.objects.filter(vendor=instance.vendor, acknowledgment_date__isnull=False).values_list('acknowledgment_date', 'issue_date')
        average_response_time = sum(abs((ack_date - issue_date).total_seconds()) for ack_date, issue_date in response_times) #/ len(response_times)
        if response_times:
            average_response_time = total_seconds / len(response_times)
        else:
            average_response_time = 0  
        instance.vendor.average_response_time = average_response_time
        instance.vendor.save()
        return Response({'acknowledgment_date': instance.acknowledgment_date})
```

## Testing or Using the API endpoints.
## what is cUrl:


## what is httpie:
HTTPie (pronounced aitch-tee-tee-pie) is a command-line HTTP client. Its goal is to make CLI interaction with web services as human-friendly as possible. HTTPie is designed for testing, debugging, and generally interacting with APIs & HTTP servers

### Install HTTPie
```bash
pip install --upgrade pip setuptools pip install --upgrade httpie

```

## Create a vendor:
### using curl (to use on command line):
curl -H "Authorization: Token your_obtained_token" -X POST http://127.0.0.1:8000/api/vendors/ -d "vendor_code=01&name=Vendor+1&contact_details=Contact+1&address=Address+1"

### using httpie:
http POST http://127.0.0.1:8000/api/vendors/ vendor_code=01 name="Vendor 1" contact_details="Contact 1" address="Address 1" "Authorization: Token your_obtained_token"
### About this API endpoint:
here this endpoint is used to create a vendor by providing the vendor details.

## List all Vendors details:
### using curl:
curl -H "Authorization: Token your_obtained_token" http://127.0.0.1:8000/api/vendors/
### using httpie:
http http://127.0.0.1:8000/api/vendors/ "Authorization: Token your_obtained_token"
### About this API endpoint:
here this endpoint is used to get the details of all vendors.
## Retrieve a specific vendor's details:
## using curl:
curl -H "Authorization: Token your_obtained_token" http://127.0.0.1:8000/api/vendors/{vendor_id}/
### using httpie:
http http://127.0.0.1:8000/api/vendors/{vendor_id}/ "Authorization: Token your_obtained_token"
### About this API endpoint:
here this endpoint is used to get the details of vendor with vendor_id which was mentioned in the command.
## Update a vendor's details:
### using curl:
#### PUT method:
curl -H "Authorization: Token your_obtained_token" -X PUT http://127.0.0.1:8000/api/vendors/{vendor_id}/ -d "vendor_code=updated code&name=Updated Vendor Name&contact_details=Updated Contact Details&address=Updated Address"
#### PATCH method:
curl -H "Authorization: Token your_obtained_token" -X PATCH http://127.0.0.1:8000/api/vendors/{vendor_id}/ -d "name=Updated Vendor Name"
### using httpie:
#### PUT method:
http PUT http://127.0.0.1:8000/api/vendors/{vendor_id}/ vendor_code="updated vendor code" name="Updated Vendor Name" contact_details="Updated Contact Details" address="Updated Address" "Authorization: Token your_obtained_token"
#### PATCH method:
http PATCH http://127.0.0.1:8000/api/vendors/{vendor_id}/ name="Updated Vendor Name" contact_details="Updated Contact Details" address="Updated Address" "Authorization: Token your_obtained_token"
### About this API endpoint:
here this endpoint we have two commands with different http methods (PUT,PATCH).As we have a primary key in the model the PUT method works as POST method (which means it creates a new vendor with given details). The PATCH method is used to update the vendor's details except vendor's id. here PUT handles updates by replacing the entire entity, so it creates a new entity. but where the PATCH handles by only updating the given fields.
## Delete a vendor:
### using curl:
curl -H "Authorization: Token your_obtained_token" -X DELETE http://127.0.0.1:8000/api/vendors/{vendor_id}/
### using httpie:
http DELETE http://127.0.0.1:8000/api/vendors/{vendor_id}/ "Authorization: Token your_obtained_token"
### About this API endpoint:
here this endpoint is used to delete the vendor with given vendor_id.
## Create a purchase_order:
### using curl:
curl -H "Authorization: Token your_obtained_token" -X POST http://127.0.0.1:8000/api/purchase_orders/ -d "po_number=01&vendor=01&order_date=2023-01-01T12:00:00&delivery_date=2023-01-10T12:00:00&items=[{"item_name":"Item 1","quantity": 10 },{"item_name": 10 }]&quality_rating=4.5&issue_date=2023-01-01T12:00:00&status=updated&acknowledgment_date=2023-01-02T12:00:00"
### using httpie:
http POST http://127.0.0.1:8000/api/purchase_orders/ po_number="01" vendor=01 order_date="2023-01-01" delivery_date="2023-01-10" items='[{"item_name": 20 }]' quality_rating=4.5 issue_date="2023-01-01" status="Pending" acknowledgment_date="2023-01-02" "Authorization: Token your_obtained_token"
### About this API endpoint:
here this endpoint is used to create a purchase_order with given details.
## List all purchase_orders details:
### using curl:
curl -H "Authorization: Token your_obtained_token" http://127.0.0.1:8000/api/purchase_orders/
### using httpie:
http http://127.0.0.1:8000/api/purchase_orders/ "Authorization: Token your_obtained_token"
### About this API endpoint:
here this endpoint is used to get the details of all purchase_orders.
## Retrieve a specific purchase_order's details:
### using curl:
curl -H "Authorization: Token your_obtained_token" http://127.0.0.1:8000/api/purchase_orders/{po_id}/
### using httpie:
http http://127.0.0.1:8000/api/purchase_orders/{po_id}/ "Authorization: Token your_obtained_token"
### About this API endpoint:
here this endpoint is used to get the details of purchase_order with given po_id.
### Update a purchase_order's details:
### using curl:
#### PUT method:
curl -H "Authorization: Token your_obtained_token" -X PUT http://127.0.0.1:8000/api/purchase_orders/{po_id}/ -d "po_number=updatedno&vendor=updatedvno&order_date=2023-01-02T12:00:00&delivery_date=2023-01-15T12:00:00&items=[{"item_name": 10 },{"item_name":10}]&quality_rating=5&issue_date=2023-01-01T12:00:00&status=updated&acknowledgment_date=2023-01-01T12:00:00"
#### PATCH method:
curl -H "Authorization: Token your_obtained_token" -X PATCH http://127.0.0.1:8000/api/purchase_orders/{po_id}/ -d "vendor=updatedvno&quality_rating=5"
#### using httpie:
#### PUT method:
http PUT http://127.0.0.1:8000/api/purchase_orders/{po_id}/ po_number="UpdatedPO001" vendor="updatedid" order_date="2023-01-02T12:00:00" delivery_date="2023-01-15T12:00:00" items:='[{"item_name": 20 }]' quality_rating:=4.8 issue_date="2023-01-01" status="updated" acknowledgment_date="2023-01-02" "Authorization: Token your_obtained_token"
#### PATCH method:
http PATCH http://127.0.0.1:8000/api/purchase_orders/{po_id}/ order_date="2023-01-02T12:00:00" delivery_date="2023-01-15T12:00:00" items:='[{"item_name": "Updated Item", "quantity": 20 }]' quality_rating:=4.8 status="updated" "Authorization: Token your_obtained_token"
### About this API endpoint:
here this endpoint we have two commands with different http methods (PUT,PATCH).As we have a primary key in the model the PUT method works as POST method (which means it creates a new purchase_order with given details). The PATCH method is used to update the purchase_order's details except purchase_order's number. here PUT handles updates by replacing the entire entity, so it creates a new entity. but where the PATCH handles by only updating the given fields.(we can provide any no of fields in PATCH mathod.)
## Delete a purchase_order:
### using curl:
curl -H "Authorization: Token your_obtained_token" -X DELETE http://127.0.0.1:8000/api/purchase_orders/{po_id}/
### using httpie:
http DELETE http://127.0.0.1:8000/api/purchase_orders/{po_id}/ "Authorization: Token your_obtained_token"
### About this API endpoint:
here this endpoint is used to delete a purchase_order with given po_id.
## Retrieve a vendor's performance metrics:
### using curl:
curl -H "Authorization: Token your_obtained_token" http://127.0.0.1:8000/api/vendors/1/performance/
### using httpie:
http http://127.0.0.1:8000/api/vendors/1/performance/ "Authorization: Token your_obtained_token"

## Update a purchase_order's details:
### using curl:
#### PUT method:
curl -H "Authorization: Token your_obtained_token" -X PUT http://127.0.0.1:8000/api/purchase_orders/{po_id}/ -d "po_number=updatedno&vendor=updatedvno&order_date=2023-01-02T12:00:00&delivery_date=2023-01-15T12:00:00&items=[{"item_name": 10 },{"item_name":10}]&quality_rating=5&issue_date=2023-01-01T12:00:00&status=updated&acknowledgment_date=2023-01-01T12:00:00"
#### PATCH method:
curl -H "Authorization: Token your_obtained_token" -X PATCH http://127.0.0.1:8000/api/purchase_orders/{po_id}/ -d "vendor=updatedvno&quality_rating=5"
### using httpie:
#### PUT method:
http PUT http://127.0.0.1:8000/api/purchase_orders/{po_id}/ po_number="UpdatedPO001" vendor="updatedid" order_date="2023-01-02T12:00:00" delivery_date="2023-01-15T12:00:00" items:='[{"item_name": 20 }]' quality_rating:=4.8 issue_date="2023-01-01" status="updated" acknowledgment_date="2023-01-02" "Authorization: Token your_obtained_token"
#### PATCH method:
http PATCH http://127.0.0.1:8000/api/purchase_orders/{po_id}/ order_date="2023-01-02T12:00:00" delivery_date="2023-01-15T12:00:00" items:='[{"item_name": "Updated Item", "quantity": 20 }]' quality_rating:=4.8 status="updated" "Authorization: Token your_obtained_token"
### About this API endpoint:
here this endpoint we have two commands with different http methods (PUT,PATCH).As we have a primary key in the model the PUT method works as POST method (which means it creates a new purchase_order with given details). The PATCH method is used to update the purchase_order's details except purchase_order's number. here PUT handles updates by replacing the entire entity, so it creates a new entity. but where the PATCH handles by only updating the given fields.(we can provide any no of fields in PATCH mathod.)
## Delete a purchase_order:
### using curl:
curl -H "Authorization: Token your_obtained_token" -X DELETE http://127.0.0.1:8000/api/purchase_orders/{po_id}/
### using httpie:
http DELETE http://127.0.0.1:8000/api/purchase_orders/{po_id}/ "Authorization: Token your_obtained_token"
### About this API endpoint:
here this endpoint is used to delete a purchase_order with given po_id.
## Retrieve a vendor's performance metrics:
### using curl:
curl -H "Authorization: Token your_obtained_token" http://127.0.0.1:8000/api/vendors/1/performance/
### using httpie:
http http://127.0.0.1:8000/api/vendors/1/performance/ "Authorization: Token your_obtained_token"
### About this API endpoint:
here this endpoint is used to retrieve the performance metrics of a vendor with given vendor_id. this performance metrics contains on_time Delivery rate, quality rating average, average response time, fulfilment rate
On time delivery rate is calculated each time a PO status changes to "completed". this is the average of no of po delivered before the delivery_date and no of total po's delivered.
quality rating average is calculated after every po completion and it is the average of all ratings given to that specific vendor.
average response time is calculated each time a po is acknowledged by the vendor. it is the time difference between issue_date and acknowledgment_date for each po, and then the average of these times for all po's of the vendor.
fulfillment rate is calculated when po status is set to "completed". this is the average of no of successfully fulfilled pos (status = "completed" without issues) by the total no of pos issued to the vendor.
## Update acknowledgment_data and trigger the recalculation of average_response_time:
### using curl:
curl -H "Authorization: Token your_obtained_token" -X PATCH http://127.0.0.1:8000/api/purchase_orders/{po_id}/acknowledge/ --data "acknowledgment_date=2023-12-30T12:00:00Z"
### using httpie:
http PATCH http://127.0.0.1:8000/api/purchase_orders/{po_id}/acknowledge/ "Authorization: Token your_obtained_token" acknowledgment_date="2023-12-20T12:00:00Z"
### About this API endpoint:
here this endpoint is used to acknowledge the purchase_order with given po_id and trigger the recalculation of average_reponse_time.