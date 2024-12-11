interface PhoneOption {
  id: string;
  value: string;
}

interface Contact {
  contactId: string;
  name: string;
  email: string;
  phoneOptions: PhoneOption[];
}

interface ComparisonResult {
  new: Contact[];
  updated: Contact[];
  nochange: Contact[];
  deleted: Contact[];
}

function compareObjectArrays(oldArray: Contact[], newArray: Contact[]): ComparisonResult {
  const result: ComparisonResult = {
    new: [],
    updated: [],
    nochange: [],
    deleted: [],
  };

  const newMap = new Map(newArray.map(item => [item.contactId, item]));

  // Check for updated, nochange, and deleted
  for (const oldItem of oldArray) {
    const newItem = newMap.get(oldItem.contactId);

    if (!newItem) {
      // If contactId is not present in newArray, it's deleted
      result.deleted.push(oldItem);
    } else {
      // Check if the contact is updated or unchanged
      const isUpdated = JSON.stringify(oldItem) !== JSON.stringify(newItem);
      if (isUpdated) {
        const updatedContact = { ...newItem };

        // Check phoneOptions for changes
        const phoneComparison = comparePhoneOptions(oldItem.phoneOptions, newItem.phoneOptions);
        updatedContact.phoneOptions = phoneComparison;

        result.updated.push(updatedContact);
      } else {
        result.nochange.push(oldItem);
      }

      // Remove the item from newMap to mark it as processed
      newMap.delete(oldItem.contactId);
    }
  }

  // Remaining items in newMap are new contacts
  result.new = Array.from(newMap.values());

  return result;
}

function comparePhoneOptions(oldOptions: PhoneOption[], newOptions: PhoneOption[]): PhoneOption[] {
  const result: PhoneOption[] = [];
  const newOptionsMap = new Map(newOptions.map(option => [option.id, option]));

  for (const oldOption of oldOptions) {
    const newOption = newOptionsMap.get(oldOption.id);
    if (!newOption) {
      // PhoneOption is deleted
      result.push({ ...oldOption, status: 'deleted' });
    } else {
      if (JSON.stringify(oldOption) !== JSON.stringify(newOption)) {
        // PhoneOption is updated
        result.push({ ...newOption, status: 'updated' });
      } else {
        // PhoneOption is unchanged
        result.push({ ...oldOption, status: 'nochange' });
      }
      newOptionsMap.delete(oldOption.id);
    }
  }

  // Remaining items in newOptionsMap are new phone options
  for (const newOption of newOptionsMap.values()) {
    result.push({ ...newOption, status: 'new' });
  }

  return result;
}
