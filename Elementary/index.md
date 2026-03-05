# Elementary

Elementary is a minimal reflection based MVVM framework.

Elementary is build with a few core principles in mind:
- Ease of use. Elementary is stupid simple and intuitive, following known C# and Unity practices with minimal domain specific knowledge.
- Ease of adoption. Elementary is fast to pick up, non destructive to existing projects, and frictionless to integrate with your existing UI.
- Ease of extension. Elementary comes with a suite of foundational tools, but every project is unique. Elementary is designed to be easily extended with custom behavior fit for your project.

These core pillars make Elementary an ideal solution for everyone from large scale projects to small/individual projects where you cannot afford to spend time onboarding a new framework, and where momentum is key!

In addition to the above, Elementary is written with nullable reference types enabled, and disabled domain reloading in mind

## Fundamentals
MVVM stands for Model-View-ViewModel. There's lots of resources out there for understanding MVVM and why you might choose it for your project, so I will just cover the basics of how it works. Your models represent your applications domain data. Your view, is the interface which displays the visual information to the user, and provides input methods to command the application logic. A UGUI Button, TextMeshProUGUI script, DragAndDrop script, CLI tool, and a screen reader are all part of the View. The ViewModel represents the visual information presented to the end user, and the set of commands the user can input. For example we may have a `Player(id, name, xp, minHealth, maxHealth, currentHealth)` as our model. We wish to display our player's name and current percent health, so our view model is `HudViewModel(playerName, healthPercent)`. Our view would consist of a UGUI Slider script for the health bar, and a TextMeshPro script for the player name.

In Elementary we use `View`s and Bindings mapped to that `View` to display our view model data in the view. A ViewModel can be bound to a `View`. Bindings map the properties of a ViewModel to other components. When a ViewModel is bound to a `View`, any `Binding`s mapped to that view are bound to their properties. Following our example from above, we could bind our `HudViewModel` to a `View`, and a `TextMeshProBinding` mapped to `playerName` will update to display that value in the `TMP_Text` script in your view. By default, this would require us to re-bind our ViewModel everytime a value changes. Which is why Elementary provides `Observable<T>` and `ObsevableList<T>`. When these values are bound, you can update them in your view model and the view will change to reflect those changes without having to rebind the entire ViewModel.

All Bindings inherit from `Binding<T>` where T is the property type the binding supports. Any type assignable to T is also bindable to a Binding of type T. A binding simply contains the instructions of what to do when the value it is bound to changes. Many bindings are provided for you, but if you have your own UI scripts, you'll have to make your own bindings. Each binding can only bind to one property. You may have to write multiple bindings for a single UI script, but this is intentional. It keeps your inspector clean and your intent clear.

A view model is any class or struct with the `ViewModel` attribute. Using value types will result in boxing, so it's encouraged to use classes. Record classes lend themselves well to view models, because it enforces immutability on the view model itself, enforcing you to use `Observable<T>` for dynamic values and think twice about when to rebind a view model vs modify it's observable properties. However part of the power of Elementary is the flexibility for you to choose what works best for you. By default, all public properties and fields show up in a Binding's inspector to be bound. You can override this behaviour with the `[NonBindable]` and `[Bindable]` attributes respectively.

## Getting Started
For the following sections we'll be using the example of an in-game shop. This is a good use case to demonstrate the basic capabilities of Elementary. This shop will consist of a list of items that can be clicked to purchase them. The shop will also have a name to display at the top of the screen.

### Creating View Models
```csharp
[ViewModel]
public record ShopItemViewModel(string Name);
[ViewModel]
public record ShopViewModel(string Name, IEnumerable<ShopItemViewModel> ShopItems);

List<ShopItemViewModel> itemViewModels = new();
foreach (var shopItem in shop) {
  ShopViewModel vm = new(shopItem.DisplayName);
  itemViewModels.Add(vm);
}

ShopViewModel vm = new(shop.DisplayName, itemViewModels);
```

### Binding A View Model To A View

<img width="918" height="168" alt="image" src="https://github.com/user-attachments/assets/e2c390ef-5ea2-49f6-9960-99d985809934" />

```csharp
View view = GetView(); // However you resolve this dependency is up to you
ShopViewModel vm = CreateViewModel(); // Similar to logic above
view.Bind(vm); // Bindings receive values from view model
view.Bind(null); // Bindings receive default values
```

### Writing A Custom Binding
Based on built-in binding provided for TextMesh

<img width="917" height="142" alt="image" src="https://github.com/user-attachments/assets/b3ea0ce3-41ee-4569-ae55-4fb353124749" />

```csharp
public class TextMeshStringTextBinding : Binding<string>
{
	[SerializeField] private TMP_Text text;

	protected override void OnValueChanged(string value, ValueSource source)
	{
		if (text == null) return;
		text.text = value;
	}
}
```

### Binding A Collection
The only provided collection binding is `TransformViewCollectionBinding`, which takes a collection of view models and spawns a template with a `View` component and binds those view models to the spawned templates.
<img width="926" height="235" alt="image" src="https://github.com/user-attachments/assets/e3ea89c3-20d0-47a1-9549-b9cd68a92d0e" />

### Receiving Input From The View
<img width="907" height="262" alt="image" src="https://github.com/user-attachments/assets/7ad01286-4f87-47e7-a845-bb227906114e" />

```csharp
// Update our view model
[ViewModel]
public record ShopItemViewModel(string Name, Command Purchase);

void OnPurchase(ShopItem item) { ... }

List<ShopItemViewModel> itemViewModels = new();
foreach (var shopItem in shop) {
  ShopViewModel vm = new(
    shopItem.DisplayName,
    () => OnPurchase(shopItem)
  );
  itemViewModels.Add(vm);
}

ShopViewModel vm = new(shop.DisplayName, itemViewModels);
view.Bind(vm);
```

### Making A Value Update Dynamically
Simply wrap your value in `Observable<T>` to make it update dynamically after being bound

```csharp
// Update our view model
[ViewModel]
public record ShopViewModel(string Name, Observable<int> CurrencyAvailable, IEnumerable<ShopItemViewModel> ShopItems);

ShopViewModel vm = CreateViewModel();
view.Bind(vm); // Displays initial values in the view model
vm.CurrencyAvailable.SetValue(CurrencyAvailable.Value + 1); // Displays new value immediately
```

### Making A Dynamic Collection
Similar to a single value, you can change your collection type to `ObservableList<T>` to make it update dynamically

```csharp
// Update our view model
[ViewModel]
public record ShopViewModel(string Name, Observable<int> CurrencyAvailable, ObservableList<ShopItemViewModel> ShopItems);

ShopViewModel vm = CreateViewModel(); // Shop Item Command Bound To OnPurchase
ShopItemViewModel itemVm = GetSomeViewModel();
vm.ShopItems.Remove(itemVm); // Item will disappear from view
vm.ShopItems.Add(itemVm); // Item will appear in view
```

## Advanced Topics
Once you've mastered the basics, these more advanced topics will really help you master Elementary

### Nesting View Models
Beyond a collection, you can also nest view models inside other view models by simply making them a property. Then in your Scene/Prefab you can use `ChildViewBinding` to bind that value to a child view. Furthermore, you can make that field `Observable<T>` where T is the child view model. Changing the observable value will result in another view model being bound to the child view. This would be useful for something like a description window, which changes the description panel depending on the item selected. Or it can simply be done to re-use view models compositionally, like a progress bar used in several UI throughout your game.

<img width="913" height="255" alt="image" src="https://github.com/user-attachments/assets/8536d156-cb60-4d2b-83d9-5422c93bbb7d" />

```csharp
[ViewModel]
public record InventoryViewModel(IEnumerable<ItemViewModel> Items, ProgressViewModel CapacityBar, Observable<DescriptionViewModel> Description);
[ViewModel]
public record ProgressViewModel(string Label, float Progress);
[ViewModel]
public record DescriptionViewModel(string ItemName, string Description);

Dictionary<Item, ItemViewModel> itemViewModels = CreateItemViewModels(items);
Dictionary<Item, DescriptionViewModel> descriptions = CreateDescriptionViewModels(items);

ProgressBarViewModel progressVm = CreateProgressViewModel();

InventoryViewModel vm = new(itemViewModels.Values, progressVm, new(null));

void OnSelectItem(Item item)
{
  vm.Description.SetValue(descriptions[item]); // Rebinds child view to new view model
}
```

### Bind Semantics
When you want to update a value in your view model, when should you rebind the entire model vs update an `Observable<T>` on the view model? This topic is called "Bind Semantics". This will become important later when we talk about transitions and FX. Ideally, you should bind/rebind a view model as a reset operation. When you are completely replacing the values and resetting the state. For example if we're viewing user profiles and we switch users, we should rebind that view. However if the user updates their username, we should probably just update an observable property. In terms of performance, binding a view model is slightly heavier as it involves light reflection on the view models at runtime. However updating an `Observable<T>` is reflection free and simply a series of event callbacks and virtual method calls.

### Transitions and FX
Most UI is dynamic, but how can we know when to apply a transition and when to immediately update the visual state? The answer is to use bind semantics. When a value changes from a view model binding, we are "resetting" the state. When an `Observable<T>` changes value, we are updating the state dynamically and should apply transitions. Using this paradigm will enable most transitions and effects you're interested in. You may have already spotted it, but `Binding<T>` exposes a `ValueSource` field which tells you whether that value change came from an observable property changing, or a view model being rebound. Additionally there are `BeforeBindingEvent` and `AfterBindingEvent` which expose unity events to enable/disable behaviours based on the semantics of the binding. You can use this to toggle transition animations without having to make a custom `Binding<T>`.

```csharp
public class AnimatedSliderViewBinding : Binding<float>
{
	[SerializeField] private UnityEngine.UI.Slider? slider;
	[SerializeField] private float animationDuration = 0.5f;

	private float targetValue;
	private Coroutine? animationCoroutine;

	protected override void OnValueChanged(float value, ValueSource source)
	{
		if (slider == null) return;
		targetValue = value;
		if (animationCoroutine == null)
		{
			animationCoroutine = StartCoroutine(AnimateSlider());
		}
	}
	private IEnumerator AnimateSlider()
	{
		if (slider == null) yield break;
		while (slider.value != targetValue)
		{
			slider.value = Mathf.MoveTowards(slider.value, targetValue, Time.deltaTime / animationDuration);
			yield return null;
		}
	}
}
```

### Writing A Custom Collection Binding
Collections have a different set of operations. For non-observable collections, only `OnCollectionSet` will be called. For `ObservableList<T>` there are several operations to help facilitate transitions and animations and managing the entities displayed in discrete steps. These operations are `OnItemAdded`, `OnItemRemoved`, `OnItemSet`, and `OnItemMoved`. Here's a very abbreviated example of what a simplified collection binding that spawns prefabs into a transform might look like. 

**Note**: The built in TransformViewCollectionBinding also has a custom editor build off the BindingEditor api, which further filters the list of bindable fields to only be a type assignable to the view in the template. By default, since it inherits from `CollectionBinding<object>` any collection could be assigned to it in the editor, and then the view would fail to bind.

```csharp
public class TransformViewCollectionBinding : CollectionBinding<object>
{
  [SerializeField] private View template;

  protected override void OnCollectionSet(IEnumerable<object> items)
  {
    DestroyAllChildren();
    foreach (object item in items)
    {
      View view = Instantiate(template, transform);
      view.Bind(item);
    }
  }

  protected override void OnItemAdded(int atIndex, object item)
  {
    View view = Instantiate(template, transform);
    view.SetSiblindIndex(atIndex);
    view.Bind(item);
  }

  protected override void OnItemRemoved(int fromIndex, object _)
  {
    View view = transform.GetChild(fromIndex).GetComponent<View>();
    view.Bind(null);
    Destroy(view.gameObject);
  }

  protected override void OnItemSet(int atIndex, object item)
  {
    View view = transform.GetChild(atIndex).GetComponent<View>();
    view.Bind(item);
  }

  protected override void OnItemMoved(int fromIndex, int toIndex, object _)
  {
    Transform child = transform.GetChild(fromIndex);
    child.SetSiblingIndex(toIndex);
  }
}
```

### Editor Scripting
This section could last forever, but if you find yourself needing to write custom editors for your custom bindings, you should inherit from `BindingEditor`. All editor scripting is done using the UIElements over IMGUI. This makes it easier to extend and easier to support more advanced behaviours. `BindingEditor` provides an API so you can get all the same base editor functionality, while still being able to change the editor for the part of your component that extends the `Binding<T>` class. For simple editors, override the `VisualElement CreateSubclassInspectorGUI()`. For more advanced editors, you can also override `VisualElement CreateBindingGUI()` to modify the UI for the Binding base fields, or you can override unity's `VisualElement CreateInspectorGUI()` to completely start from scratch. Lastly, a convenience method is provided to add your own filtering to the available options to be bound. Simply override `IEnumerable<ViewModelMemberInfo> FilterBindingPathOptions(IEnumerable<ViewModelMemberInfo> options)` and return the filtered options.

There are a ton of utilities provided and publicly accessible for your convenience. One important one is ViewModelMemberInfo. This class represents a member on a view model class, and has information such as whether it's observable, it's raw type, and it's bindable type. For example an `Observable<T>` will have a binding type of `T`.

For a good example, see `TransforViewCollectionBindingEditor.cs`
