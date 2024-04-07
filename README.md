# vuci tasks

Implementations of various tasks to learn about **Vue.js** and **uci** interopability. UI was designed using **Ant Design** UI component library.

## components app

This app is designed to manage a list of hardware interfaces. It provides a simple web UI interface for adding, editing, and deleting HW interfaces. **Vuci** components were used to display a list of records.

## overview app

This app is designed to view quick summaries of various information about device. Information is shown in card form. The order of cards can be rearranged. It is also possible to hide selected cards. Information is loaded dynamically which means that data to a card is provided through a prop object. This object can also have additional properties which are used to tell card component on how data should be displayed.
