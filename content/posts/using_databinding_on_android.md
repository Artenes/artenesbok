---
title: "Using Data Binding on android"
date: 2020-02-05T18:46:07-04:00
draft: false
toc: false
images:
tags: 
  - android
  - databinding
---

Let's use [data binding](https://developer.android.com/topic/libraries/data-binding?hl=pt-br) in an Android application.

1.Enabling data binding in your build.gradle file.

````groovy
android {
  dataBinding {
    enabled = true
  }
}
````

2.Using data binding in your xml layout file.

- Wrap it with the ``<layout>`` tag.
- Use the ``<data>`` tag to decalre the variables used in this layout.

````xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>
        <!-- We are using ViewMode here as an example, but it can be any data type -->
        <variable
            name="viewModel"
            type="package.MyViewModel" />
    </data>

    <!-- Your layout here -->

</layout>
````

3.Inflating the layout in your Activity.

````kotlin
import androidx.appcompat.app.AppCompatActivity

class AccountViewActivity : AppCompatActivity() { 

  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    val viewModel = ViewModelProviders.of(this, factory).get(MyViewModel::class.java)

    val binding = DataBindingUtil.setContentView<ActivityAccountViewBinding>(this,R.layout.my_view)
    binding.lifecycleOwner = this //this is to use LiveData
    binding.viewModel = viewModel
  }

}
````

4.Exposing data from your ViewModel to the View.

You have to make your data ``MutableLiveDatas`` so the data binding library can write and read data from/to it.

````kotlin
class MyViewModel() : ViewModel() {

  //it can be of any type
  val myData = MutableLiveData<String>()

}
````

5.Binding data from your ViewModel to the View.

Pay attention to use ``@={}`` to bind the data in the view.

````xml
<com.google.android.material.textfield.TextInputEditText
    android:id="@+id/textInputName"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:text="@={viewModel.data}" />
````

6.Binding data to a custom view.

Data binding does not know how to bind data to custom views, so you have to tell it through ``BidingAdapters``.

````kotlin
import androidx.databinding.BindingAdapter
import androidx.databinding.InverseBindingAdapter
import androidx.databinding.InverseBindingListener

object Bindings {

    //this method tells how to set the value in the custom view
    //this myCustomProperty can be any name
    //the method also can be any name
    //data binding will find any method with this annotation
    @BindingAdapter("myCustomProperty")
    @JvmStatic
    fun setValue(view: MyCustomView, newValue: String?) {
        //this is to avoid infinite loops
        if (view.value != newValue) {
            view.value = newValue ?: ""
        }
    }

    //this method tells how to fetch data from the custom view
    @InverseBindingAdapter(attribute = "myCustomProperty")
    @JvmStatic
    fun getValue(view: MyCystomView): String {
        return view.value
    }

    //this is a standard, you need to take the name and add the sufix PropertyAttrChanged
    //this is how we tell when the data changes in behalve of the user
    @BindingAdapter("app:myCustomPropertyAttrChanged")
    @JvmStatic
    fun setListener(view: CustomView, attrChange: InverseBindingListener) {
        //you create this listener
        view.listener = object:MyCustomView.OnValueChangeListener {
            override fun onValueChanged(newValue: String) {
                attrChange.onChange()
            }
        }
    }

}
````

7.Adding onClick event in buttons with data binding

````xml
<Button
  android:id="@+id/buttonSave"
  android:layout_width="match_parent"
  android:layout_height="wrap_content"
  android:text="@string/save"
  android:onClick="@{() -> viewModel.save()}"/>
````