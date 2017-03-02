# ng-store

```typescript
abstract class Storable {
  public id: string;
  public $path: string[];
}
```

```typescript
interface Store {
  set(path: string[], value: any): Promise<void>;
  remove(path: string[]): Promise<void>;
  attach(path: string[], value: any): Promise<void>;
  detach(path: string[], value: any): Promise<void>;
  get<TValue>(path: string[]): Observable<TValue>;
  watch<TValue>(path: string[]): Observable<TValue>;
}
```


```typescript
@Component({
  selector: 'order-list-comp',
  template: `<order-comp 
                (click)="selectOrder(order)" 
                [order]="order" 
                [selected]="order === selectedOrder$ | async" 
                *ngFor="let order of orders$ | async"></order-comp>`
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
  }
  
  public addOrder(order: Order): void {
    this.store.attach(['orders'], order);
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
  
  public add(): void {
    this.sidenav.open().then(item => {
      if(item)
        this.store.attach([order, 'items'], item);
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
  
  public changeQuantity(quantity: number): void {
    this.store.set([item, 'quantity'], quantity);
  }
}
```
