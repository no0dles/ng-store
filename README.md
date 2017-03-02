# ng-store

```typescript
abstract class StoreItem {
  public _id: string;
  public _path: string[];
  
  constructor() {
    this._id = Guid.new();
  }
}
```

```typescript
interface Options {
  deep: number; // default => 0, recursive => -1
}
```

```typescript
interface Store {
  set<TValue>(path: string[], value: TValue): Promise<TValue>;
  remove(path: string[]): Promise<any>;
  attach<TValue>(path: string[], value: TValue): Promise<TValue>;
  detach<TValue>(path: string[], value: TValue): Promise<TValue>;
  get<TValue>(path: string[], options?: Options): Promise<TValue>;
  watch<TValue>(path: string[]): Observable<TValue>;
}
```


```typescript
class Order extends StoreItem {
  public number: string;
  public items: OrderItem[] = [];
}
```

```typescript
class OrderItem extends StoreItem {
  public quantity: number;
}
```


```typescript
@Component({
  selector: 'order-list-comp',
  template: `
    <div class="actions">
      <button (click)="addOrder()">Add</button>
    </div>
    <div class="orders">
      <order-comp 
          (click)="selectOrder(order)" 
          [order]="order" 
          [selected]="order === selectedOrder$ | async" 
          *ngFor="let order of orders$ | async"></order-comp>
    </div>
  `
})
class OrderListComponent {
  public orders$: Observable<Order[]>;
  public selectedOrder$: Observable<Order>;
  
  constructor(private store: Store) {
    this.orders$ = this.store.watch<Order[]>(['orders']);
    this.selectedOrder$ = this.store.watch<Order>(['selectedOrder']);
  }
  
  public selectOrder(order: Order): void {
    this.store.set(['selectedOrder'], order);
    // set order for key ['selectedOrder']
  }
  
  public addOrder(): void {
    this.store.attach(['orders'], new Order());
  }
  
  public removeOrder(order: Order): void {
    this.store.detach(['orders'], order);
  }
}
```


```typescript
@Component({
  selector: 'order-comp',
  template: `
    <div class="actions">
      <button (click)="add()">Add</button>
    </div>
    <div class="order-items">
      <order-item-comp [item]="item" *ngFor="let item of items$ | async"></order-item-comp>
    </div>
  `
})
class OrderComponent {
  @Input() public order: Order;
  @Input() public selected: boolean;
  
  public items$: Observable<OrderItem[]>;
  
  constructor(private store: Store) {
    this.items$ = this.store.watch<OrderItem[]>([order, 'items']);
  }
  
  ngOnInit() {
    console.log(this.order);
    // { _id: "order-guid", items: null, _path: ["orders", "order-guid"] }
  }
  
  public add(): void {
    this.sidenav.open().then(item => {
      if(item)
        this.store.attach([order, 'items'], item);
        // attach item object for key ["orders", "order-guid", "items"]
    });
  }
}
```


```typescript
@Component({
  selector: 'order-item-comp',
  template: `
    <label for="name">Name></label>
    <span id="name">{{item.name}}</span>
  
    <label for="quantity">Quantity></label>
    <input id="quantity" type="number" (ngModelChanged)="changeQuantity($event)" />
  `
})
class OrderItemComponent {
  @Input() public item: OrderItem;
  
  constructor(private store: Store) {
    
  }
  
  ngOnInit() {
    console.log(this.item);
    // { _id: "item-guid", quantity: 0, _path: ["orders", "order-guid", "items", "item-guid"] }
  }
  
  public changeQuantity(quantity: number): void {
    this.store.set([item, 'quantity'], quantity);
    // set quantity for key ["orders", "order-guid", "items", "item-guid", "quantity"]
  }
}
```
