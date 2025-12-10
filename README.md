import React, { useState } from "react";
import { Menu, ShoppingCart, Plus, Edit2, Trash2, X } from "lucide-react";

// Simple Logo Component
function Logo() {
  return (
    <div className="flex items-center gap-2">
      <div className="w-8 h-8 bg-red-500 rounded-lg flex items-center justify-center">
        <span className="text-white font-bold text-xl">R</span>
      </div>
      <span className="font-bold text-xl">Restaurant</span>
    </div>
  );
}

// Add Category Component
function AddCategory({ onAddCategory }) {
  const [categoryName, setCategoryName] = useState("");
  
  const handleAddCategory = () => {
    if (categoryName.trim()) {
      onAddCategory(categoryName.trim());
      setCategoryName("");
    }
  };
  
  return (
    <div className="flex gap-3">
      <input
        type="text"
        value={categoryName}
        onChange={(e) => setCategoryName(e.target.value)}
        onKeyPress={(e) => e.key === 'Enter' && handleAddCategory()}
        placeholder="Enter category name"
        className="flex-1 px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-red-500 focus:border-transparent"
      />
      <button
        onClick={handleAddCategory}
        className="px-6 py-2 bg-red-500 text-white rounded-lg font-medium hover:bg-red-600 transition-all"
      >
        Add Category
      </button>
    </div>
  );
}

export default function AdminFoodPage() {
  const [activeCategory, setActiveCategory] = useState("All Dishes");
  const [showModal, setShowModal] = useState(false);
  const [editingDish, setEditingDish] = useState(null);
  const [formData, setFormData] = useState({
    name: "",
    price: "",
    description: "",
    image: "",
    category: "Pizzas",
  });

  const [categories, setCategories] = useState([
    { name: "All Dishes", count: 0, isDeletable: false },
    { name: "Pizzas", count: 0, isDeletable: true },
    { name: "Burgers", count: 0, isDeletable: true },
    { name: "Salads", count: 0, isDeletable: true },
    { name: "Brunch", count: 0, isDeletable: true }
  ]);

  const [allDishes, setAllDishes] = useState([]);

  const handleReset = () => {
    if (window.confirm("Are you sure you want to reset everything? All dishes and custom categories will be deleted.")) {
      // Reset to initial state
      setAllDishes([]);
      setCategories([
        { name: "All Dishes", count: 0, isDeletable: false },
        { name: "Pizzas", count: 0, isDeletable: true },
        { name: "Burgers", count: 0, isDeletable: true },
        { name: "Salads", count: 0, isDeletable: true },
        { name: "Brunch", count: 0, isDeletable: true }
      ]);
      setActiveCategory("All Dishes");
    }
  };

  const handleAddCategory = (categoryName) => {
    setCategories([...categories, { 
      name: categoryName, 
      count: 0, 
      isDeletable: true 
    }]);
  };

  const handleDeleteCategory = (categoryName) => {
    if (window.confirm(`Are you sure you want to delete the category "${categoryName}"?`)) {
      // Delete the category
      setCategories(categories.filter(cat => cat.name !== categoryName));
      
      // Delete all dishes in this category
      setAllDishes(allDishes.filter(dish => dish.category !== categoryName));
      
      // If we're viewing the deleted category, switch to "All Dishes"
      if (activeCategory === categoryName) {
        setActiveCategory("All Dishes");
      }
    }
  };

  const handleDrop = (files) => {
    if (files.length > 0) {
      const file = files[0];
      if (file.type.startsWith("image/")) {
        const reader = new FileReader();
        reader.onload = (event) => {
          setFormData((prev) => ({ ...prev, image: event.target.result }));
        };
        reader.readAsDataURL(file);
      } else {
        alert("Please select an image file.");
      }
    }
  };

  const getFilteredDishes = (categoryName) => {
    if (categoryName === "All Dishes") {
      return allDishes;
    }
    return allDishes.filter((dish) => dish.category === categoryName);
  };

  const getCategoryCount = (categoryName) => {
    if (categoryName === "All Dishes") {
      return allDishes.length;
    }
    return allDishes.filter((dish) => dish.category === categoryName).length;
  };

  const handleAddNew = (category) => {
    setEditingDish(null);
    const defaultCategory = categories.find(c => c.name !== "All Dishes")?.name || "Pizzas";
    setFormData({
      name: "",
      price: "",
      description: "",
      image: "",
      category: category === "All Dishes" ? defaultCategory : category,
    });
    setShowModal(true);
  };

  const handleEdit = (dish) => {
    setEditingDish(dish);
    setFormData({
      name: dish.name,
      price: dish.price.toString(),
      description: dish.description,
      image: dish.image,
      category: dish.category,
    });
    setShowModal(true);
  };

  const handleDelete = (id) => {
    if (window.confirm("Are you sure to delete?")) {
      setAllDishes(allDishes.filter((dish) => dish.id !== id));
    }
  };

  const handleSubmit = () => {
    if (
      !formData.name ||
      !formData.price ||
      !formData.description ||
      !formData.image
    ) {
      alert("Refill all the section!!!");
      return;
    }

    if (editingDish) {
      setAllDishes(
        allDishes.map((dish) =>
          dish.id === editingDish.id
            ? { ...dish, ...formData, price: parseFloat(formData.price) }
            : dish
        )
      );
    } else {
      const newDish = {
        id: Date.now(),
        ...formData,
        price: parseFloat(formData.price),
      };
      setAllDishes([...allDishes, newDish]);
    }

    setShowModal(false);
  };

  const displayedDishes = getFilteredDishes(activeCategory);

  return (
    <div className="flex min-h-screen bg-gray-50">
      {/* Sidebar */}
      <div className="w-64 bg-white shadow-lg p-6">
        <div className="flex items-center gap-3 mb-8">
          <Logo />
        </div>

        <nav className="space-y-2">
          <button className="flex items-center gap-3 w-full px-4 py-3 bg-gray-900 text-white rounded-xl font-medium cursor-pointer">
            <Menu size={20} />
            Food menu
          </button>
          <button className="flex items-center gap-3 w-full px-4 py-3 text-gray-600 hover:bg-gray-100 rounded-xl font-medium cursor-pointer">
            <ShoppingCart size={20} />
            Orders
          </button>
        </nav>
      </div>

      <div className="flex-1 p-8">
        <div className="flex justify-end items-center mb-6">
          <div className="w-10 h-10 bg-purple-500 rounded-full overflow-hidden cursor-pointer flex items-center justify-center text-white font-bold">
            A
          </div>
        </div>

        {/* Categories Management Section */}
        <div className="bg-white rounded-2xl p-6 mb-8 border-2">
          <div className="flex justify-between items-center mb-6">
            <h2 className="text-2xl font-bold text-gray-800">
              Dishes category
            </h2>
            <button
              onClick={handleReset}
              className="px-4 py-2 bg-red-500 text-white rounded-lg font-medium hover:bg-red-600 transition-all flex items-center gap-2"
            >
              <X size={18} />
              Reset All
            </button>
          </div>
          <AddCategory onAddCategory={handleAddCategory} />
          
          {/* Category List */}
          <div className="mt-6 space-y-2">
            {categories.filter(cat => cat.name !== "All Dishes").map((cat) => (
              <div
                key={cat.name}
                className="flex items-center justify-between p-4 bg-gray-50 rounded-lg hover:bg-gray-100 transition-all"
              >
                <div className="flex items-center gap-3">
                  <div className="w-2 h-2 bg-red-500 rounded-full"></div>
                  <span className="font-medium text-gray-800">{cat.name}</span>
                  <span className="text-sm text-gray-500">
                    ({getCategoryCount(cat.name)} dishes)
                  </span>
                </div>
                <button
                  onClick={() => handleDeleteCategory(cat.name)}
                  className="p-2 hover:bg-red-50 rounded-lg transition-all group"
                  title="Delete category"
                >
                  <Trash2 size={18} className="text-gray-400 group-hover:text-red-500" />
                </button>
              </div>
            ))}
          </div>
        </div>

        {/* Category Filter Tabs */}
        <div className="flex gap-3 mb-6 overflow-x-auto pb-2">
          {categories.map((cat) => (
            <button
              key={cat.name}
              onClick={() => setActiveCategory(cat.name)}
              className={`px-6 py-2 rounded-lg font-medium whitespace-nowrap transition-all ${
                activeCategory === cat.name
                  ? "bg-red-500 text-white shadow-lg"
                  : "bg-white text-gray-600 hover:bg-gray-100"
              }`}
            >
              {cat.name} ({getCategoryCount(cat.name)})
            </button>
          ))}
        </div>

        {/* Dishes Section */}
        <div className="mb-12">
          <h3 className="text-2xl font-bold text-gray-800 mb-6">
            {activeCategory} ({displayedDishes.length})
          </h3>
          <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
            {/* Add New Card */}
            <div
              onClick={() => handleAddNew(activeCategory)}
              className="border-2 border-dashed border-gray-300 rounded-2xl p-8 flex flex-col items-center justify-center min-h-[280px] hover:border-red-400 hover:bg-red-50 transition-all cursor-pointer"
            >
              <div className="w-16 h-16 rounded-full bg-red-500 text-white flex items-center justify-center mb-4">
                <Plus size={28} />
              </div>
              <p className="text-gray-600 font-medium text-center">
                Add new Dish to
                <br />
                {activeCategory}
              </p>
            </div>

            {/* Dish Cards */}
            {displayedDishes.map((dish) => (
              <div
                key={dish.id}
                className="bg-white rounded-2xl shadow-md overflow-hidden hover:shadow-xl transition-all"
              >
                <div className="relative">
                  <img
                    src={dish.image}
                    alt={dish.name}
                    className="w-full h-48 object-cover"
                  />
                  <div className="absolute top-3 right-3 flex gap-2">
                    <button
                      onClick={() => handleEdit(dish)}
                      className="w-10 h-10 bg-white rounded-full flex items-center justify-center shadow-lg hover:bg-blue-50"
                    >
                      <Edit2 size={18} className="text-blue-500" />
                    </button>
                    <button
                      onClick={() => handleDelete(dish.id)}
                      className="w-10 h-10 bg-white rounded-full flex items-center justify-center shadow-lg hover:bg-red-50"
                    >
                      <Trash2 size={18} className="text-red-500" />
                    </button>
                  </div>
                </div>
                <div className="p-4">
                  <div className="flex justify-between items-start mb-2">
                    <h4 className="text-red-500 font-semibold text-lg">
                      {dish.name}
                    </h4>
                    <span className="font-bold text-gray-800">
                      ${dish.price}
                    </span>
                  </div>
                  <p className="text-gray-600 text-sm">{dish.description}</p>
                  <div className="mt-2">
                    <span className="inline-block px-3 py-1 bg-gray-100 text-gray-600 text-xs rounded-full">
                      {dish.category}
                    </span>
                  </div>
                </div>
              </div>
            ))}
          </div>
        </div>
      </div>

      {showModal && (
        <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 p-4">
          <div className="bg-white rounded-2xl p-8 max-w-md w-full max-h-[90vh] overflow-y-auto">
            <div className="flex justify-between items-center mb-6">
              <h3 className="text-2xl font-bold text-gray-800">
                {editingDish ? "Хоол засах" : "Add new food"}
              </h3>
              <button
                onClick={() => setShowModal(false)}
                className="w-8 h-8 rounded-full hover:bg-gray-100 flex items-center justify-center"
              >
                <X size={20} />
              </button>
            </div>

            <div className="space-y-4">
              <div>
                <label className="block text-sm font-medium text-gray-700 mb-1">
                  Name
                </label>
                <input
                  type="text"
                  value={formData.name}
                  onChange={(e) =>
                    setFormData({ ...formData, name: e.target.value })
                  }
                  className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-red-500 focus:border-transparent"
                  placeholder="Food name"
                />
              </div>

              <div>
                <label className="block text-sm font-medium text-gray-700 mb-1">
                  Price ($)
                </label>
                <input
                  type="number"
                  step="0.01"
                  value={formData.price}
                  onChange={(e) =>
                    setFormData({ ...formData, price: e.target.value })
                  }
                  className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-red-500 focus:border-transparent"
                  placeholder="12.99"
                />
              </div>

              <div>
                <label className="block text-sm font-medium text-gray-700 mb-1">
                  Description
                </label>
                <textarea
                  value={formData.description}
                  onChange={(e) =>
                    setFormData({ ...formData, description: e.target.value })
                  }
                  className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-red-500 focus:border-transparent"
                  rows={3}
                  placeholder="Food description"
                />
              </div>

              <div>
                <label className="block text-sm font-medium text-gray-700 mb-1">
                  Image
                </label>

                {formData.image && (
                  <div className="relative mb-4">
                    <img
                      src={formData.image}
                      alt="Preview"
                      className="w-full h-40 object-cover rounded-lg border-2"
                    />
                    <button
                      onClick={() =>
                        setFormData((prev) => ({ ...prev, image: "" }))
                      }
                      className="absolute top-2 right-2 w-7 h-7 bg-red-500 text-white rounded-full flex items-center justify-center hover:bg-red-600 transition-all"
                    >
                      <X size={16} />
                    </button>
                  </div>
                )}

                {!formData.image && (
                  <label
                    htmlFor="file-upload"
                    onDrop={(e) => {
                      e.preventDefault();
                      const droppedFiles = Array.from(e.dataTransfer.files);
                      handleDrop(droppedFiles);
                    }}
                    onDragOver={(e) => {
                      e.preventDefault();
                    }}
                    className="border-2 border-dashed bg-gray-100 h-[106px] border-gray-300 rounded-lg p-4 text-center cursor-pointer hover:border-gray-400 transition-colors flex items-center justify-center"
                  >
                    <div className="flex flex-col items-center gap-2">
                      <div className="rounded-full bg-white p-2">
                        <svg
                          xmlns="http://www.w3.org/2000/svg"
                          width="13"
                          height="13"
                          viewBox="0 0 13 13"
                          fill="none"
                        >
                          <path
                            d="M12.5 8.5L10.4427 6.44267C10.1926 6.19271 9.85355 6.05229 9.5 6.05229C9.14645 6.05229 8.80737 6.19271 8.55733 6.44267L2.5 12.5M1.83333 0.5H11.1667C11.903 0.5 12.5 1.09695 12.5 1.83333V11.1667C12.5 11.903 11.903 12.5 11.1667 12.5H1.83333C1.09695 12.5 0.5 11.903 0.5 11.1667V1.83333C0.5 1.09695 1.09695 0.5 1.83333 0.5ZM5.83333 4.5C5.83333 5.23638 5.23638 5.83333 4.5 5.83333C3.76362 5.83333 3.16667 5.23638 3.16667 4.5C3.16667 3.76362 3.76362 3.16667 4.5 3.16667C5.23638 3.16667 5.83333 3.76362 5.83333 4.5Z"
                            stroke="#09090B"
                            strokeLinecap="round"
                            strokeLinejoin="round"
                          />
                        </svg>
                      </div>
                      <p className="text-sm text-gray-600">
                        Choose a file or drag & drop it here
                      </p>
                    </div>
                  </label>
                )}

                <input
                  type="file"
                  accept="image/*"
                  onChange={(e) => {
                    if (e.target.files) {
                      handleDrop(Array.from(e.target.files));
                    }
                  }}
                  className="hidden"
                  id="file-upload"
                />
              </div>

              <div>
                <label className="block text-sm font-medium text-gray-700 mb-1">
                  Sections
                </label>
                <select
                  value={formData.category}
                  onChange={(e) =>
                    setFormData({ ...formData, category: e.target.value })
                  }
                  className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-red-500 focus:border-transparent"
                >
                  {categories
                    .filter((c) => c.name !== "All Dishes")
                    .map((cat) => (
                      <option key={cat.name} value={cat.name}>
                        {cat.name}
                      </option>
                    ))}
                </select>
              </div>

              <div className="flex gap-3 pt-4">
                <button
                  onClick={() => setShowModal(false)}
                  className="flex-1 px-4 py-3 bg-gray-200 text-gray-700 rounded-lg font-medium hover:bg-gray-300 transition-all"
                >
                  Цуцлах
                </button>
                <button
                  onClick={handleSubmit}
                  className="flex-1 px-4 py-3 bg-red-500 text-white rounded-lg font-medium hover:bg-red-600 transition-all"
                >
                  {editingDish ? "Save" : "Add"}
                </button>
              </div>
            </div>
          </div>
        </div>
      )}
    </div>
  );
}
