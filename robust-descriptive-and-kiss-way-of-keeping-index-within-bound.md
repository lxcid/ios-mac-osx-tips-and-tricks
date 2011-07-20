# Robust, Descriptive and KISS Way of Keeping Index Within Bound #

## Issue ##
Often when creating a navigation user interface, we will have to deal with a list of items and manage their selection state. This is a fairly common pattern and one not too difficult to accomplish. But such a common and simple pattern are frequently written in longer than expected line of codes. Then again, this doesn't justify the description we given it earlier.

Well, the additional codes come from the need to guard the code against index out of bounds error. We generally use a variable to track the selected index while keeping a list of items. Obviously the selected index have to be within the bound of the list. How would you write it?

We can drop bounds guard everywhere and things will just work. My challenge to you is to write it in the least amount of code. If you actually follow me up until here, I believe you are someone who care about code quality and I am going to share with you my solution.

## My take ##
The key is to clamp the selected index within the bound upon setting it. Clamp function generally expand to a series of if else statements, but a more descriptive way to describe the behavior of clamp is this.

	void set_selected_index(int the_selected_index) {
		int lower_bound = 0;
		int upper_bound = MAX(lower_bound, total_number_of_items_in_list - 1);
		the_selected_index = MAX(lower_bound, MIN(the_selected_index, upper_bound));
		if (selected_index != the_selected_index) {
			selected_index = the_selected_index;
			set_need_update();
		}
	}

The first 3 line effectively clamp `the_selected_index` within the bound, follow by a guard to update and notify the need to update only if there is changes.

This function is extremely robust and can take quite a beating, e.g:

	void next() {
		set_selected_index(selected_index + 1);
	}
	
	void previous() {
		set_selected_index(selected_index - 1);
	}

The above 2 functions will generally be bind to controls for a user to tap on. No matter how many times the user tap on both buttons, the user will not get out of bound `selected_index` and only update appropriately when there are changes.

Neat huh?