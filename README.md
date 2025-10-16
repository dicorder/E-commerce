# E-commerce
Web Develpoment


import React, { useState, useEffect } from "react";
import { base44 } from "@/api/base44Client";
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { Button } from "@/components/ui/button";
import { Link } from "react-router-dom";
import { createPageUrl } from "@/utils";
import { ArrowRight, Sparkles, ShoppingBag, TrendingUp } from "lucide-react";
import ProductCard from "../components/products/ProductCard";
import { motion } from "framer-motion";

export default function Home() {
  const [user, setUser] = useState(null);
  const queryClient = useQueryClient();

  useEffect(() => {
    base44.auth.me().then(setUser).catch(() => setUser(null));
  }, []);

  const { data: featuredProducts = [] } = useQuery({
    queryKey: ['featured-products'],
    queryFn: () => base44.entities.Product.filter({ featured: true }, '-created_date', 6),
  });

  const addToCartMutation = useMutation({
    mutationFn: async (product) => {
      if (!user) {
        base44.auth.redirectToLogin();
        return;
      }
      const existingItems = await base44.entities.CartItem.filter({
        user_email: user.email,
        product_id: product.id
      });
      
      if (existingItems.length > 0) {
        await base44.entities.CartItem.update(existingItems[0].id, {
          quantity: (existingItems[0].quantity || 1) + 1
        });
      } else {
        await base44.entities.CartItem.create({
          product_id: product.id,
          product_name: product.name,
          product_price: product.price,
          product_image: product.image_url,
          quantity: 1,
          user_email: user.email
        });
      }
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['cart'] });
    },
  });

  return (
    <div>
      {/* Hero Section */}
      <section className="relative bg-gradient-to-br from-purple-50 via-white to-pink-50 py-20">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
          <div className="grid lg:grid-cols-2 gap-12 items-center">
            <motion.div
              initial={{ opacity: 0, x: -50 }}
              animate={{ opacity: 1, x: 0 }}
              transition={{ duration: 0.6 }}
            >
              <div className="flex items-center gap-2 mb-4">
                <Sparkles className="w-5 h-5 text-amber-500" />
                <span className="text-sm font-medium text-purple-600">New Arrivals</span>
              </div>
              <h1 className="text-5xl md:text-6xl font-bold text-gray-900 mb-6">
                Discover Your Perfect
                <span className="bg-gradient-to-r from-purple-600 to-pink-600 bg-clip-text text-transparent"> Style</span>
              </h1>
              <p className="text-xl text-gray-600 mb-8">
                Shop the latest trends and timeless classics. Quality products, unbeatable prices.
              </p>
              <div className="flex flex-col sm:flex-row gap-4">
                <Link to={createPageUrl("Products")}>
                  <Button size="lg" className="bg-purple-600 hover:bg-purple-700 w-full sm:w-auto">
                    Shop Now
                    <ArrowRight className="ml-2 w-4 h-4" />
                  </Button>
                </Link>
                <Link to={createPageUrl("Products") + "?category=featured"}>
                  <Button size="lg" variant="outline" className="w-full sm:w-auto">
                    View Collections
                  </Button>
                </Link>
              </div>
            </motion.div>
            <motion.div
              initial={{ opacity: 0, scale: 0.9 }}
              animate={{ opacity: 1, scale: 1 }}
              transition={{ duration: 0.6, delay: 0.2 }}
              className="relative"
            >
              <img
                src="https://images.unsplash.com/photo-1441986300917-64674bd600d8?w=800"
                alt="Shopping"
                className="rounded-2xl shadow-2xl"
              />
              <div className="absolute -bottom-6 -left-6 bg-white p-6 rounded-xl shadow-lg">
                <div className="flex items-center gap-3">
                  <div className="w-12 h-12 bg-purple-100 rounded-full flex items-center justify-center">
                    <ShoppingBag className="w-6 h-6 text-purple-600" />
                  </div>
                  <div>
                    <p className="text-2xl font-bold text-gray-900">5000+</p>
                    <p className="text-sm text-gray-600">Products</p>
                  </div>
                </div>
              </div>
            </motion.div>
          </div>
        </div>
      </section>

      {/* Features */}
      <section className="py-16 bg-white">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
          <div className="grid md:grid-cols-3 gap-8">
            {[
              { icon: TrendingUp, title: "Best Prices", desc: "Competitive pricing on all products" },
              { icon: ShoppingBag, title: "Fast Delivery", desc: "Quick shipping to your doorstep" },
              { icon: Sparkles, title: "Quality Guarantee", desc: "100% satisfaction guaranteed" }
            ].map((feature, idx) => (
              <motion.div
                key={idx}
                initial={{ opacity: 0, y: 20 }}
                animate={{ opacity: 1, y: 0 }}
                transition={{ delay: idx * 0.1 }}
                className="text-center p-6 rounded-xl hover:bg-purple-50 transition-colors"
              >
                <div className="w-16 h-16 bg-purple-100 rounded-full flex items-center justify-center mx-auto mb-4">
                  <feature.icon className="w-8 h-8 text-purple-600" />
                </div>
                <h3 className="text-xl font-semibold mb-2">{feature.title}</h3>
                <p className="text-gray-600">{feature.desc}</p>
              </motion.div>
            ))}
          </div>
        </div>
      </section>

      {/* Featured Products */}
      <section className="py-16 bg-gray-50">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
          <div className="flex justify-between items-center mb-8">
            <div>
              <h2 className="text-3xl font-bold text-gray-900 mb-2">Featured Products</h2>
              <p className="text-gray-600">Handpicked favorites just for you</p>
            </div>
            <Link to={createPageUrl("Products")}>
              <Button variant="outline">
                View All
                <ArrowRight className="ml-2 w-4 h-4" />
              </Button>
            </Link>
          </div>
          <div className="grid sm:grid-cols-2 lg:grid-cols-3 gap-6">
            {featuredProducts.map((product) => (
              <ProductCard
                key={product.id}
                product={product}
                onAddToCart={(p) => addToCartMutation.mutate(p)}
                isAdding={addToCartMutation.isPending}
              />
            ))}
          </div>
        </div>
      </section>
    </div>
  );
}       
import React, { useState, useEffect } from "react";
import { base44 } from "@/api/base44Client";
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { Input } from "@/components/ui/input";
import { Button } from "@/components/ui/button";
import { Search, SlidersHorizontal } from "lucide-react";
import FilterSidebar from "../components/products/FilterSidebar";
import ProductCard from "../components/products/ProductCard";
import { Skeleton } from "@/components/ui/skeleton";
import { Sheet, SheetContent, SheetTrigger } from "@/components/ui/sheet";

export default function Products() {
  const [user, setUser] = useState(null);
  const [searchQuery, setSearchQuery] = useState("");
  const [filters, setFilters] = useState({
    categories: [],
    priceRange: [0, 1000],
    inStock: false
  });
  const queryClient = useQueryClient();

  useEffect(() => {
    base44.auth.me().then(setUser).catch(() => setUser(null));
  }, []);

  const { data: allProducts = [], isLoading } = useQuery({
    queryKey: ['products'],
    queryFn: () => base44.entities.Product.list('-created_date'),
  });

  const filteredProducts = allProducts.filter(product => {
    const matchesSearch = product.name?.toLowerCase().includes(searchQuery.toLowerCase()) ||
                         product.description?.toLowerCase().includes(searchQuery.toLowerCase());
    const matchesCategory = filters.categories.length === 0 || filters.categories.includes(product.category);
    const matchesPrice = product.price >= filters.priceRange[0] && product.price <= filters.priceRange[1];
    const matchesStock = !filters.inStock || product.stock > 0;
    
    return matchesSearch && matchesCategory && matchesPrice && matchesStock;
  });

  const addToCartMutation = useMutation({
    mutationFn: async (product) => {
      if (!user) {
        base44.auth.redirectToLogin();
        return;
      }
      const existingItems = await base44.entities.CartItem.filter({
        user_email: user.email,
        product_id: product.id
      });
      
      if (existingItems.length > 0) {
        await base44.entities.CartItem.update(existingItems[0].id, {
          quantity: (existingItems[0].quantity || 1) + 1
        });
      } else {
        await base44.entities.CartItem.create({
          product_id: product.id,
          product_name: product.name,
          product_price: product.price,
          product_image: product.image_url,
          quantity: 1,
          user_email: user.email
        });
      }
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['cart'] });
    },
  });

  const clearFilters = () => {
    setFilters({
      categories: [],
      priceRange: [0, 1000],
      inStock: false
    });
  };

  return (
    <div className="min-h-screen bg-gray-50">
      <div className="bg-white border-b">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
          <h1 className="text-4xl font-bold text-gray-900 mb-6">Our Products</h1>
          <div className="flex gap-4">
            <div className="relative flex-1">
              <Search className="absolute left-3 top-1/2 -translate-y-1/2 w-5 h-5 text-gray-400" />
              <Input
                placeholder="Search products..."
                value={searchQuery}
                onChange={(e) => setSearchQuery(e.target.value)}
                className="pl-10"
              />
            </div>
            <Sheet>
              <SheetTrigger asChild>
                <Button variant="outline" className="lg:hidden">
                  <SlidersHorizontal className="w-4 h-4 mr-2" />
                  Filters
                </Button>
              </SheetTrigger>
              <SheetContent side="left">
                <FilterSidebar
                  filters={filters}
                  onFilterChange={setFilters}
                  onClearFilters={clearFilters}
                />
              </SheetContent>
            </Sheet>
          </div>
        </div>
      </div>

      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
        <div className="flex gap-8">
          <aside className="hidden lg:block w-64 flex-shrink-0">
            <div className="sticky top-20">
              <FilterSidebar
                filters={filters}
                onFilterChange={setFilters}
                onClearFilters={clearFilters}
              />
            </div>
          </aside>

          <div className="flex-1">
            <div className="flex justify-between items-center mb-6">
              <p className="text-gray-600">
                {filteredProducts.length} products found
              </p>
            </div>

            {isLoading ? (
              <div className="grid sm:grid-cols-2 lg:grid-cols-3 gap-6">
                {Array(6).fill(0).map((_, i) => (
                  <div key={i} className="space-y-4">
                    <Skeleton className="aspect-square w-full" />
                    <Skeleton className="h-4 w-3/4" />
                    <Skeleton className="h-4 w-1/2" />
                  </div>
                ))}
              </div>
            ) : filteredProducts.length === 0 ? (
              <div className="text-center py-12">
                <p className="text-gray-500 text-lg">No products found</p>
                <Button onClick={clearFilters} className="mt-4">
                  Clear Filters
                </Button>
              </div>
            ) : (
              <div className="grid sm:grid-cols-2 lg:grid-cols-3 gap-6">
                {filteredProducts.map((product) => (
                  <ProductCard
                    key={product.id}
                    product={product}
                    onAddToCart={(p) => addToCartMutation.mutate(p)}
                    isAdding={addToCartMutation.isPending}
                  />
                ))}
              </div>
            )}
          </div>
        </div>
      </div>
    </div>
  );
}
import React, { useState, useEffect } from "react";
import { base44 } from "@/api/base44Client";
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { Button } from "@/components/ui/button";
import { Badge } from "@/components/ui/badge";
import { Card } from "@/components/ui/card";
import { ShoppingCart, Check, Minus, Plus, Package, Truck, Shield } from "lucide-react";
import { useNavigate } from "react-router-dom";
import { createPageUrl } from "@/utils";
import { Skeleton } from "@/components/ui/skeleton";

export default function ProductDetail() {
  const navigate = useNavigate();
  const [user, setUser] = useState(null);
  const [quantity, setQuantity] = useState(1);
  const queryClient = useQueryClient();
  
  const urlParams = new URLSearchParams(window.location.search);
  const productId = urlParams.get('id');

  useEffect(() => {
    base44.auth.me().then(setUser).catch(() => setUser(null));
  }, []);

  const { data: product, isLoading } = useQuery({
    queryKey: ['product', productId],
    queryFn: async () => {
      const products = await base44.entities.Product.filter({ id: productId });
      return products[0];
    },
    enabled: !!productId,
  });

  const addToCartMutation = useMutation({
    mutationFn: async () => {
      if (!user) {
        base44.auth.redirectToLogin();
        return;
      }
      const existingItems = await base44.entities.CartItem.filter({
        user_email: user.email,
        product_id: product.id
      });
      
      if (existingItems.length > 0) {
        await base44.entities.CartItem.update(existingItems[0].id, {
          quantity: (existingItems[0].quantity || 1) + quantity
        });
      } else {
        await base44.entities.CartItem.create({
          product_id: product.id,
          product_name: product.name,
          product_price: product.price,
          product_image: product.image_url,
          quantity,
          user_email: user.email
        });
      }
      navigate(createPageUrl("Cart"));
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['cart'] });
    },
  });

  if (isLoading) {
    return (
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-12">
        <div className="grid lg:grid-cols-2 gap-12">
          <Skeleton className="aspect-square w-full rounded-xl" />
          <div className="space-y-4">
            <Skeleton className="h-8 w-3/4" />
            <Skeleton className="h-6 w-1/4" />
            <Skeleton className="h-24 w-full" />
            <Skeleton className="h-10 w-full" />
          </div>
        </div>
      </div>
    );
  }

  if (!product) {
    return (
      <div className="max-w-7xl mx-auto px-4 py-12 text-center">
        <h2 className="text-2xl font-bold mb-4">Product not found</h2>
        <Button onClick={() => navigate(createPageUrl("Products"))}>
          Back to Products
        </Button>
      </div>
    );
  }

  return (
    <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-12">
      <div className="grid lg:grid-cols-2 gap-12">
        <div className="relative aspect-square rounded-xl overflow-hidden bg-gray-100">
          <img
            src={product.image_url || 'https://images.unsplash.com/photo-1505740420928-5e560c06d30e?w=800'}
            alt={product.name}
            className="w-full h-full object-cover"
          />
          {product.featured && (
            <Badge className="absolute top-4 right-4 bg-amber-500">
              Featured
            </Badge>
          )}
        </div>

        <div>
          <Badge variant="outline" className="mb-4">
            {product.category}
          </Badge>
          <h1 className="text-4xl font-bold text-gray-900 mb-4">
            {product.name}
          </h1>
          <div className="flex items-center gap-4 mb-6">
            <span className="text-4xl font-bold text-purple-600">
              ${product.price?.toFixed(2)}
            </span>
            {product.stock > 0 ? (
              <Badge className="bg-green-100 text-green-800">
                <Check className="w-3 h-3 mr-1" />
                In Stock ({product.stock} available)
              </Badge>
            ) : (
              <Badge variant="destructive">Out of Stock</Badge>
            )}
          </div>

          <p className="text-gray-600 mb-8 text-lg leading-relaxed">
            {product.description}
          </p>

          {product.stock > 0 && (
            <Card className="p-6 mb-6">
              <div className="flex items-center gap-4 mb-4">
                <span className="font-medium">Quantity:</span>
                <div className="flex items-center gap-2">
                  <Button
                    variant="outline"
                    size="icon"
                    onClick={() => setQuantity(Math.max(1, quantity - 1))}
                  >
                    <Minus className="w-4 h-4" />
                  </Button>
                  <span className="w-12 text-center font-semibold">{quantity}</span>
                  <Button
                    variant="outline"
                    size="icon"
                    onClick={() => setQuantity(Math.min(product.stock, quantity + 1))}
                  >
                    <Plus className="w-4 h-4" />
                  </Button>
                </div>
              </div>
              <Button
                onClick={() => addToCartMutation.mutate()}
                disabled={addToCartMutation.isPending}
                className="w-full bg-purple-600 hover:bg-purple-700"
                size="lg"
              >
                <ShoppingCart className="w-5 h-5 mr-2" />
                Add to Cart - ${(product.price * quantity).toFixed(2)}
              </Button>
            </Card>
          )}

          <div className="grid md:grid-cols-3 gap-4">
            <Card className="p-4 text-center">
              <Truck className="w-6 h-6 mx-auto mb-2 text-purple-600" />
              <p className="text-sm font-medium">Free Shipping</p>
            </Card>
            <Card className="p-4 text-center">
              <Package className="w-6 h-6 mx-auto mb-2 text-purple-600" />
              <p className="text-sm font-medium">Easy Returns</p>
            </Card>
            <Card className="p-4 text-center">
              <Shield className="w-6 h-6 mx-auto mb-2 text-purple-600" />
              <p className="text-sm font-medium">Secure Payment</p>
            </Card>
          </div>
        </div>
      </div>
    </div>
  );
}

import React, { useState, useEffect } from "react";
import { base44 } from "@/api/base44Client";
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { Button } from "@/components/ui/button";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Trash2, Plus, Minus, ShoppingCart, ArrowRight } from "lucide-react";
import { useNavigate } from "react-router-dom";
import { createPageUrl } from "@/utils";

export default function Cart() {
  const [user, setUser] = useState(null);
  const navigate = useNavigate();
  const queryClient = useQueryClient();

  useEffect(() => {
    base44.auth.me().then(setUser).catch(() => {
      base44.auth.redirectToLogin();
    });
  }, []);

  const { data: cartItems = [], isLoading } = useQuery({
    queryKey: ['cart', user?.email],
    queryFn: () => user ? base44.entities.CartItem.filter({ user_email: user.email }) : [],
    enabled: !!user,
  });

  const updateQuantityMutation = useMutation({
    mutationFn: ({ itemId, quantity }) => base44.entities.CartItem.update(itemId, { quantity }),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['cart'] });
    },
  });

  const removeItemMutation = useMutation({
    mutationFn: (itemId) => base44.entities.CartItem.delete(itemId),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['cart'] });
    },
  });

  const total = cartItems.reduce((sum, item) => sum + (item.product_price * item.quantity), 0);

  if (!user) return null;

  if (isLoading) {
    return (
      <div className="max-w-7xl mx-auto px-4 py-12">
        <p>Loading cart...</p>
      </div>
    );
  }

  if (cartItems.length === 0) {
    return (
      <div className="max-w-7xl mx-auto px-4 py-12 text-center">
        <ShoppingCart className="w-24 h-24 mx-auto mb-6 text-gray-300" />
        <h2 className="text-3xl font-bold mb-4">Your cart is empty</h2>
        <p className="text-gray-600 mb-8">Start shopping to add items to your cart!</p>
        <Button
          onClick={() => navigate(createPageUrl("Products"))}
          className="bg-purple-600 hover:bg-purple-700"
        >
          Browse Products
        </Button>
      </div>
    );
  }

  return (
    <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-12">
      <h1 className="text-4xl font-bold text-gray-900 mb-8">Shopping Cart</h1>
      
      <div className="grid lg:grid-cols-3 gap-8">
        <div className="lg:col-span-2 space-y-4">
          {cartItems.map((item) => (
            <Card key={item.id}>
              <CardContent className="p-6">
                <div className="flex gap-4">
                  <img
                    src={item.product_image || 'https://images.unsplash.com/photo-1505740420928-5e560c06d30e?w=200'}
                    alt={item.product_name}
                    className="w-24 h-24 object-cover rounded-lg"
                  />
                  <div className="flex-1">
                    <h3 className="font-semibold text-lg mb-2">{item.product_name}</h3>
                    <p className="text-2xl font-bold text-purple-600 mb-4">
                      ${item.product_price?.toFixed(2)}
                    </p>
                    <div className="flex items-center gap-4">
                      <div className="flex items-center gap-2">
                        <Button
                          variant="outline"
                          size="icon"
                          onClick={() => updateQuantityMutation.mutate({
                            itemId: item.id,
                            quantity: Math.max(1, item.quantity - 1)
                          })}
                        >
                          <Minus className="w-4 h-4" />
                        </Button>
                        <span className="w-12 text-center font-semibold">{item.quantity}</span>
                        <Button
                          variant="outline"
                          size="icon"
                          onClick={() => updateQuantityMutation.mutate({
                            itemId: item.id,
                            quantity: item.quantity + 1
                          })}
                        >
                          <Plus className="w-4 h-4" />
                        </Button>
                      </div>
                      <Button
                        variant="ghost"
                        size="icon"
                        onClick={() => removeItemMutation.mutate(item.id)}
                      >
                        <Trash2 className="w-4 h-4 text-red-500" />
                      </Button>
                    </div>
                  </div>
                  <div className="text-right">
                    <p className="text-sm text-gray-500 mb-1">Subtotal</p>
                    <p className="text-xl font-bold">
                      ${(item.product_price * item.quantity).toFixed(2)}
                    </p>
                  </div>
                </div>
              </CardContent>
            </Card>
          ))}
        </div>

        <div>
          <Card className="sticky top-20">
            <CardHeader>
              <CardTitle>Order Summary</CardTitle>
            </CardHeader>
            <CardContent className="space-y-4">
              <div className="flex justify-between text-gray-600">
                <span>Subtotal ({cartItems.length} items)</span>
                <span>${total.toFixed(2)}</span>
              </div>
              <div className="flex justify-between text-gray-600">
                <span>Shipping</span>
                <span className="text-green-600">Free</span>
              </div>
              <div className="border-t pt-4 flex justify-between text-xl font-bold">
                <span>Total</span>
                <span className="text-purple-600">${total.toFixed(2)}</span>
              </div>
              <Button
                onClick={() => navigate(createPageUrl("Checkout"))}
                className="w-full bg-purple-600 hover:bg-purple-700"
                size="lg"
              >
                Proceed to Checkout
                <ArrowRight className="ml-2 w-5 h-5" />
              </Button>
            </CardContent>
          </Card>
        </div>
      </div>
    </div>
  );
}
import React, { useState, useEffect } from "react";
import { base44 } from "@/api/base44Client";
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { Button } from "@/components/ui/button";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { useNavigate } from "react-router-dom";
import { createPageUrl } from "@/utils";
import { CreditCard, Lock } from "lucide-react";

export default function Checkout() {
  const [user, setUser] = useState(null);
  const [isProcessing, setIsProcessing] = useState(false);
  const navigate = useNavigate();
  const queryClient = useQueryClient();

  const [shippingInfo, setShippingInfo] = useState({
    full_name: "",
    address: "",
    city: "",
    postal_code: "",
    country: ""
  });

  useEffect(() => {
    base44.auth.me().then(u => {
      setUser(u);
      setShippingInfo(prev => ({
        ...prev,
        full_name: u.full_name || ""
      }));
    }).catch(() => {
      base44.auth.redirectToLogin();
    });
  }, []);

  const { data: cartItems = [] } = useQuery({
    queryKey: ['cart', user?.email],
    queryFn: () => user ? base44.entities.CartItem.filter({ user_email: user.email }) : [],
    enabled: !!user,
  });

  const total = cartItems.reduce((sum, item) => sum + (item.product_price * item.quantity), 0);

  const placeOrderMutation = useMutation({
    mutationFn: async () => {
      const orderItems = cartItems.map(item => ({
        product_name: item.product_name,
        quantity: item.quantity,
        price: item.product_price
      }));

      await base44.entities.Order.create({
        user_email: user.email,
        items: orderItems,
        total_amount: total,
        status: "pending",
        shipping_address: shippingInfo,
        payment_status: "completed"
      });

      for (const item of cartItems) {
        await base44.entities.CartItem.delete(item.id);
      }
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['cart'] });
      queryClient.invalidateQueries({ queryKey: ['orders'] });
      navigate(createPageUrl("Orders"));
    },
  });

  const handleSubmit = async (e) => {
    e.preventDefault();
    setIsProcessing(true);
    await placeOrderMutation.mutate();
    setIsProcessing(false);
  };

  if (!user) return null;

  if (cartItems.length === 0) {
    navigate(createPageUrl("Cart"));
    return null;
  }

  return (
    <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-12">
      <h1 className="text-4xl font-bold text-gray-900 mb-8">Checkout</h1>

      <div className="grid lg:grid-cols-3 gap-8">
        <div className="lg:col-span-2">
          <form onSubmit={handleSubmit}>
            <Card className="mb-6">
              <CardHeader>
                <CardTitle>Shipping Information</CardTitle>
              </CardHeader>
              <CardContent className="space-y-4">
                <div>
                  <Label htmlFor="full_name">Full Name</Label>
                  <Input
                    id="full_name"
                    value={shippingInfo.full_name}
                    onChange={(e) => setShippingInfo({...shippingInfo, full_name: e.target.value})}
                    required
                  />
                </div>
                <div>
                  <Label htmlFor="address">Address</Label>
                  <Input
                    id="address"
                    value={shippingInfo.address}
                    onChange={(e) => setShippingInfo({...shippingInfo, address: e.target.value})}
                    required
                  />
                </div>
                <div className="grid md:grid-cols-2 gap-4">
                  <div>
                    <Label htmlFor="city">City</Label>
                    <Input
                      id="city"
                      value={shippingInfo.city}
                      onChange={(e) => setShippingInfo({...shippingInfo, city: e.target.value})}
                      required
                    />
                  </div>
                  <div>
                    <Label htmlFor="postal_code">Postal Code</Label>
                    <Input
                      id="postal_code"
                      value={shippingInfo.postal_code}
                      onChange={(e) => setShippingInfo({...shippingInfo, postal_code: e.target.value})}
                      required
                    />
                  </div>
                </div>
                <div>
                  <Label htmlFor="country">Country</Label>
                  <Input
                    id="country"
                    value={shippingInfo.country}
                    onChange={(e) => setShippingInfo({...shippingInfo, country: e.target.value})}
                    required
                  />
                </div>
              </CardContent>
            </Card>

            <Card>
              <CardHeader>
                <CardTitle className="flex items-center gap-2">
                  <CreditCard className="w-5 h-5" />
                  Payment Information
                </CardTitle>
              </CardHeader>
              <CardContent>
                <div className="bg-blue-50 border border-blue-200 rounded-lg p-4 mb-4">
                  <p className="text-sm text-blue-800">
                    <Lock className="w-4 h-4 inline mr-1" />
                    This is a demo checkout. No real payment will be processed.
                  </p>
                </div>
                <Button
                  type="submit"
                  disabled={isProcessing}
                  className="w-full bg-purple-600 hover:bg-purple-700"
                  size="lg"
                >
                  {isProcessing ? "Processing..." : `Place Order - $${total.toFixed(2)}`}
                </Button>
              </CardContent>
            </Card>
          </form>
        </div>

        <div>
          <Card className="sticky top-20">
            <CardHeader>
              <CardTitle>Order Summary</CardTitle>
            </CardHeader>
            <CardContent className="space-y-4">
              {cartItems.map((item) => (
                <div key={item.id} className="flex justify-between text-sm">
                  <span>{item.product_name} x{item.quantity}</span>
                  <span>${(item.product_price * item.quantity).toFixed(2)}</span>
                </div>
              ))}
              <div className="border-t pt-4 flex justify-between text-xl font-bold">
                <span>Total</span>
                <span className="text-purple-600">${total.toFixed(2)}</span>
              </div>
            </CardContent>
          </Card>
        </div>
      </div>
    </div>
  );
}
import React, { useState, useEffect } from "react";
import { base44 } from "@/api/base44Client";
import { useQuery } from "@tanstack/react-query";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";
import { Package, Clock, Truck, CheckCircle, XCircle } from "lucide-react";
import { format } from "date-fns";

const statusIcons = {
  pending: Clock,
  processing: Package,
  shipped: Truck,
  delivered: CheckCircle,
  cancelled: XCircle
};

const statusColors = {
  pending: "bg-yellow-100 text-yellow-800",
  processing: "bg-blue-100 text-blue-800",
  shipped: "bg-purple-100 text-purple-800",
  delivered: "bg-green-100 text-green-800",
  cancelled: "bg-red-100 text-red-800"
};

export default function Orders() {
  const [user, setUser] = useState(null);

  useEffect(() => {
    base44.auth.me().then(setUser).catch(() => {
      base44.auth.redirectToLogin();
    });
  }, []);

  const { data: orders = [], isLoading } = useQuery({
    queryKey: ['orders', user?.email],
    queryFn: () => user ? base44.entities.Order.filter({ user_email: user.email }, '-created_date') : [],
    enabled: !!user,
  });

  if (!user) return null;

  return (
    <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-12">
      <h1 className="text-4xl font-bold text-gray-900 mb-8">My Orders</h1>

      {isLoading ? (
        <p>Loading orders...</p>
      ) : orders.length === 0 ? (
        <Card>
          <CardContent className="py-12 text-center">
            <Package className="w-16 h-16 mx-auto mb-4 text-gray-300" />
            <h3 className="text-xl font-semibold mb-2">No orders yet</h3>
            <p className="text-gray-600">Start shopping to see your orders here!</p>
          </CardContent>
        </Card>
      ) : (
        <div className="space-y-6">
          {orders.map((order) => {
            const StatusIcon = statusIcons[order.status];
            return (
              <Card key={order.id}>
                <CardHeader>
                  <div className="flex justify-between items-start">
                    <div>
                      <CardTitle className="mb-2">
                        Order #{order.id.slice(0, 8).toUpperCase()}
                      </CardTitle>
                      <p className="text-sm text-gray-500">
                        Placed on {format(new Date(order.created_date), "MMMM d, yyyy")}
                      </p>
                    </div>
                    <Badge className={statusColors[order.status]}>
                      <StatusIcon className="w-3 h-3 mr-1" />
                      {order.status}
                    </Badge>
                  </div>
                </CardHeader>
                <CardContent>
                  <div className="space-y-3 mb-4">
                    {order.items?.map((item, idx) => (
                      <div key={idx} className="flex justify-between text-sm">
                        <span>{item.product_name} x{item.quantity}</span>
                        <span className="font-medium">${(item.price * item.quantity).toFixed(2)}</span>
                      </div>
                    ))}
                  </div>
                  <div className="border-t pt-4 flex justify-between items-center">
                    <div className="text-sm text-gray-600">
                      {order.shipping_address && (
                        <p>
                          Shipping to: {order.shipping_address.city}, {order.shipping_address.country}
                        </p>
                      )}
                    </div>
                    <div className="text-right">
                      <p className="text-sm text-gray-500">Total</p>
                      <p className="text-2xl font-bold text-purple-600">
                        ${order.total_amount?.toFixed(2)}
                      </p>
                    </div>
                  </div>
                </CardContent>
              </Card>
            );
          })}
        </div>
      )}
    </div>
  );
}
import React, { useState, useEffect } from "react";
import { base44 } from "@/api/base44Client";
import { useQuery } from "@tanstack/react-query";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { FolderKanban, CheckSquare, Clock, TrendingUp } from "lucide-react";
import { Link } from "react-router-dom";
import { createPageUrl } from "@/utils";
import { BarChart, Bar, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer, PieChart, Pie, Cell } from "recharts";
import { format, startOfWeek, endOfWeek } from "date-fns";

const COLORS = ['#6366F1', '#8B5CF6', '#EC4899', '#F59E0B'];

export default function Dashboard() {
  const [user, setUser] = useState(null);

  useEffect(() => {
    base44.auth.me().then(setUser).catch(() => base44.auth.redirectToLogin());
  }, []);

  const { data: projects = [] } = useQuery({
    queryKey: ['projects'],
    queryFn: () => base44.entities.Project.list('-created_date'),
    initialData: [],
  });

  const { data: allTasks = [] } = useQuery({
    queryKey: ['all-tasks'],
    queryFn: () => base44.entities.Task.list('-created_date'),
    initialData: [],
  });

  const myTasks = allTasks.filter(task => task.assigned_to === user?.email);
  const activeProjects = projects.filter(p => p.status === 'active').length;
  const completedTasks = myTasks.filter(t => t.status === 'done').length;
  const pendingTasks = myTasks.filter(t => t.status !== 'done').length;

  const tasksByStatus = [
    { name: 'To Do', value: allTasks.filter(t => t.status === 'todo').length },
    { name: 'In Progress', value: allTasks.filter(t => t.status === 'in_progress').length },
    { name: 'Review', value: allTasks.filter(t => t.status === 'review').length },
    { name: 'Done', value: allTasks.filter(t => t.status === 'done').length },
  ];

  const projectsByStatus = [
    { name: 'Planning', value: projects.filter(p => p.status === 'planning').length },
    { name: 'Active', value: projects.filter(p => p.status === 'active').length },
    { name: 'On Hold', value: projects.filter(p => p.status === 'on_hold').length },
    { name: 'Completed', value: projects.filter(p => p.status === 'completed').length },
  ];

  return (
    <div className="p-6 space-y-6">
      <div>
        <h1 className="text-3xl font-bold text-gray-900">Dashboard</h1>
        <p className="text-gray-500 mt-1">Welcome back! Here's your project overview.</p>
      </div>

      <div className="grid md:grid-cols-2 lg:grid-cols-4 gap-6">
        <Card>
          <CardHeader className="flex flex-row items-center justify-between pb-2">
            <CardTitle className="text-sm font-medium text-gray-600">
              Total Projects
            </CardTitle>
            <FolderKanban className="w-5 h-5 text-indigo-600" />
          </CardHeader>
          <CardContent>
            <div className="text-3xl font-bold">{projects.length}</div>
            <p className="text-xs text-gray-500 mt-1">
              {activeProjects} active
            </p>
          </CardContent>
        </Card>

        <Card>
          <CardHeader className="flex flex-row items-center justify-between pb-2">
            <CardTitle className="text-sm font-medium text-gray-600">
              My Tasks
            </CardTitle>
            <CheckSquare className="w-5 h-5 text-blue-600" />
          </CardHeader>
          <CardContent>
            <div className="text-3xl font-bold">{myTasks.length}</div>
            <p className="text-xs text-gray-500 mt-1">
              {completedTasks} completed
            </p>
          </CardContent>
        </Card>

        <Card>
          <CardHeader className="flex flex-row items-center justify-between pb-2">
            <CardTitle className="text-sm font-medium text-gray-600">
              Pending Tasks
            </CardTitle>
            <Clock className="w-5 h-5 text-orange-600" />
          </CardHeader>
          <CardContent>
            <div className="text-3xl font-bold">{pendingTasks}</div>
            <p className="text-xs text-gray-500 mt-1">
              Need attention
            </p>
          </CardContent>
        </Card>

        <Card>
          <CardHeader className="flex flex-row items-center justify-between pb-2">
            <CardTitle className="text-sm font-medium text-gray-600">
              Completion Rate
            </CardTitle>
            <TrendingUp className="w-5 h-5 text-green-600" />
          </CardHeader>
          <CardContent>
            <div className="text-3xl font-bold">
              {myTasks.length > 0 ? Math.round((completedTasks / myTasks.length) * 100) : 0}%
            </div>
            <p className="text-xs text-gray-500 mt-1">
              Overall progress
            </p>
          </CardContent>
        </Card>
      </div>

      <div className="grid lg:grid-cols-2 gap-6">
        <Card>
          <CardHeader>
            <CardTitle>Tasks by Status</CardTitle>
          </CardHeader>
          <CardContent>
            <ResponsiveContainer width="100%" height={300}>
              <BarChart data={tasksByStatus}>
                <CartesianGrid strokeDasharray="3 3" />
                <XAxis dataKey="name" />
                <YAxis />
                <Tooltip />
                <Bar dataKey="value" fill="#6366F1" />
              </BarChart>
            </ResponsiveContainer>
          </CardContent>
        </Card>

        <Card>
          <CardHeader>
            <CardTitle>Projects by Status</CardTitle>
          </CardHeader>
          <CardContent>
            <ResponsiveContainer width="100%" height={300}>
              <PieChart>
                <Pie
                  data={projectsByStatus}
                  cx="50%"
                  cy="50%"
                  labelLine={false}
                  label={({ name, value }) => `${name}: ${value}`}
                  outerRadius={100}
                  fill="#8884d8"
                  dataKey="value"
                >
                  {projectsByStatus.map((entry, index) => (
                    <Cell key={`cell-${index}`} fill={COLORS[index % COLORS.length]} />
                  ))}
                </Pie>
                <Tooltip />
              </PieChart>
            </ResponsiveContainer>
          </CardContent>
        </Card>
      </div>

      <Card>
        <CardHeader>
          <CardTitle>Recent Activity</CardTitle>
        </CardHeader>
        <CardContent>
          <div className="space-y-3">
            {myTasks.slice(0, 5).map((task) => (
              <div key={task.id} className="flex items-center justify-between p-3 bg-gray-50 rounded-lg">
                <div>
                  <p className="font-medium">{task.title}</p>
                  <p className="text-sm text-gray-500">
                    {task.status.replace('_', ' ')} • {format(new Date(task.created_date), 'MMM d')}
                  </p>
                </div>
                <Link to={`${createPageUrl("ProjectDetail")}?id=${task.project_id}`}>
                  <span className="text-sm text-indigo-600 hover:underline">View Project</span>
                </Link>
              </div>
            ))}
          </div>
        </CardContent>
      </Card>
    </div>
  );
}
import React, { useState, useEffect } from "react";
import { base44 } from "@/api/base44Client";
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Search } from "lucide-react";
import ProjectCard from "../components/projects/ProjectCard";
import CreateProjectDialog from "../components/projects/CreateProjectDialog";
import { Tabs, TabsList, TabsTrigger } from "@/components/ui/tabs";

export default function Projects() {
  const [user, setUser] = useState(null);
  const [searchQuery, setSearchQuery] = useState("");
  const [statusFilter, setStatusFilter] = useState("all");
  const queryClient = useQueryClient();

  useEffect(() => {
    base44.auth.me().then(setUser).catch(() => base44.auth.redirectToLogin());
  }, []);

  const { data: projects = [], isLoading } = useQuery({
    queryKey: ['projects'],
    queryFn: () => base44.entities.Project.list('-created_date'),
    initialData: [],
  });

  const { data: allTasks = [] } = useQuery({
    queryKey: ['all-tasks'],
    queryFn: () => base44.entities.Task.list(),
    initialData: [],
  });

  const createProjectMutation = useMutation({
    mutationFn: (projectData) => base44.entities.Project.create(projectData),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['projects'] });
    },
  });

  const filteredProjects = projects.filter(project => {
    const matchesSearch = project.name?.toLowerCase().includes(searchQuery.toLowerCase());
    const matchesStatus = statusFilter === "all" || project.status === statusFilter;
    return matchesSearch && matchesStatus;
  });

  const getProjectTaskCounts = (projectId) => {
    const projectTasks = allTasks.filter(task => task.project_id === projectId);
    const completed = projectTasks.filter(task => task.status === 'done').length;
    return { total: projectTasks.length, completed };
  };

  return (
    <div className="p-6 space-y-6">
      <div className="flex flex-col md:flex-row justify-between items-start md:items-center gap-4">
        <div>
          <h1 className="text-3xl font-bold text-gray-900">Projects</h1>
          <p className="text-gray-500 mt-1">Manage and track all your projects</p>
        </div>
        <CreateProjectDialog
          onCreateProject={(data) => createProjectMutation.mutate(data)}
          isCreating={createProjectMutation.isPending}
        />
      </div>

      <div className="flex flex-col md:flex-row gap-4">
        <div className="relative flex-1">
          <Search className="absolute left-3 top-1/2 -translate-y-1/2 w-5 h-5 text-gray-400" />
          <Input
            placeholder="Search projects..."
            value={searchQuery}
            onChange={(e) => setSearchQuery(e.target.value)}
            className="pl-10"
          />
        </div>
        <Tabs value={statusFilter} onValueChange={setStatusFilter}>
          <TabsList>
            <TabsTrigger value="all">All</TabsTrigger>
            <TabsTrigger value="active">Active</TabsTrigger>
            <TabsTrigger value="planning">Planning</TabsTrigger>
            <TabsTrigger value="completed">Completed</TabsTrigger>
          </TabsList>
        </Tabs>
      </div>

      {isLoading ? (
        <div>Loading projects...</div>
      ) : filteredProjects.length === 0 ? (
        <div className="text-center py-12">
          <p className="text-gray-500 text-lg">No projects found</p>
        </div>
      ) : (
        <div className="grid sm:grid-cols-2 lg:grid-cols-3 gap-6">
          {filteredProjects.map((project) => {
            const { total, completed } = getProjectTaskCounts(project.id);
            return (
              <ProjectCard
                key={project.id}
                project={project}
                tasksCount={total}
                completedTasks={completed}
              />
            );
          })}
        </div>
      )}
    </div>
  );
}
import React, { useState, useEffect } from "react";
import { base44 } from "@/api/base44Client";
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { Button } from "@/components/ui/button";
import { Badge } from "@/components/ui/badge";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Plus, ArrowLeft, Calendar, Users } from "lucide-react";
import { useNavigate } from "react-router-dom";
import { createPageUrl } from "@/utils";
import { format } from "date-fns";
import TaskBoard from "../components/tasks/TaskBoard";
import CreateTaskDialog from "../components/tasks/CreateTaskDialog";

export default function ProjectDetail() {
  const [user, setUser] = useState(null);
  const [createTaskOpen, setCreateTaskOpen] = useState(false);
  const navigate = useNavigate();
  const queryClient = useQueryClient();
  
  const urlParams = new URLSearchParams(window.location.search);
  const projectId = urlParams.get('id');

  useEffect(() => {
    base44.auth.me().then(setUser).catch(() => base44.auth.redirectToLogin());
  }, []);

  const { data: project, isLoading } = useQuery({
    queryKey: ['project', projectId],
    queryFn: async () => {
      const projects = await base44.entities.Project.filter({ id: projectId });
      return projects[0];
    },
    enabled: !!projectId,
  });

  const { data: tasks = [] } = useQuery({
    queryKey: ['tasks', projectId],
    queryFn: () => base44.entities.Task.filter({ project_id: projectId }),
    enabled: !!projectId,
  });

  const createTaskMutation = useMutation({
    mutationFn: (taskData) => base44.entities.Task.create(taskData),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['tasks'] });
    },
  });

  const updateTaskMutation = useMutation({
    mutationFn: ({ taskId, status }) => base44.entities.Task.update(taskId, { status }),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['tasks'] });
    },
  });

  const handleTaskMove = (taskId, newStatus) => {
    updateTaskMutation.mutate({ taskId, status: newStatus });
  };

  if (isLoading) {
    return <div className="p-6">Loading project...</div>;
  }

  if (!project) {
    return (
      <div className="p-6 text-center">
        <h2 className="text-2xl font-bold mb-4">Project not found</h2>
        <Button onClick={() => navigate(createPageUrl("Projects"))}>
          Back to Projects
        </Button>
      </div>
    );
  }

  const completedTasks = tasks.filter(t => t.status === 'done').length;
  const progress = tasks.length > 0 ? (completedTasks / tasks.length) * 100 : 0;

  return (
    <div className="p-6 space-y-6">
      <div className="flex items-center gap-4">
        <Button
          variant="outline"
          size="icon"
          onClick={() => navigate(createPageUrl("Projects"))}
        >
          <ArrowLeft className="w-4 h-4" />
        </Button>
        <div className="flex-1">
          <div className="flex items-center gap-3 mb-2">
            <div 
              className="w-4 h-4 rounded-full" 
              style={{ backgroundColor: project.color || '#6366F1' }}
            />
            <h1 className="text-3xl font-bold text-gray-900">{project.name}</h1>
          </div>
          <p className="text-gray-600">{project.description}</p>
        </div>
      </div>

      <div className="grid md:grid-cols-4 gap-4">
        <Card>
          <CardHeader className="pb-2">
            <CardTitle className="text-sm font-medium text-gray-600">Status</CardTitle>
          </CardHeader>
          <CardContent>
            <Badge className="text-sm">{project.status}</Badge>
          </CardContent>
        </Card>

        <Card>
          <CardHeader className="pb-2">
            <CardTitle className="text-sm font-medium text-gray-600">Priority</CardTitle>
          </CardHeader>
          <CardContent>
            <Badge variant="outline" className="text-sm">{project.priority}</Badge>
          </CardContent>
        </Card>

        <Card>
          <CardHeader className="pb-2">
            <CardTitle className="text-sm font-medium text-gray-600">Progress</CardTitle>
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold">{Math.round(progress)}%</div>
            <p className="text-xs text-gray-500">{completedTasks}/{tasks.length} tasks</p>
          </CardContent>
        </Card>

        <Card>
          <CardHeader className="pb-2">
            <CardTitle className="text-sm font-medium text-gray-600">Deadline</CardTitle>
          </CardHeader>
          <CardContent>
            {project.end_date ? (
              <div className="flex items-center gap-1">
                <Calendar className="w-4 h-4 text-gray-500" />
                <span className="text-sm">{format(new Date(project.end_date), 'MMM d, yyyy')}</span>
              </div>
            ) : (
              <span className="text-sm text-gray-500">No deadline</span>
            )}
          </CardContent>
        </Card>
      </div>

      <div className="flex justify-between items-center">
        <h2 className="text-2xl font-bold">Task Board</h2>
        <Button
          onClick={() => setCreateTaskOpen(true)}
          className="bg-indigo-600 hover:bg-indigo-700"
        >
          <Plus className="w-4 h-4 mr-2" />
          Add Task
        </Button>
      </div>

      <TaskBoard
        tasks={tasks}
        onTaskClick={(task) => {}}
        onTaskMove={handleTaskMove}
      />

      <CreateTaskDialog
        open={createTaskOpen}
        onOpenChange={setCreateTaskOpen}
        projectId={projectId}
        onCreateTask={(data) => createTaskMutation.mutate(data)}
        isCreating={createTaskMutation.isPending}
      />
    </div>
  );
}

import React, { useState, useEffect } from "react";
import { base44 } from "@/api/base44Client";
import { useQuery } from "@tanstack/react-query";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";
import { Calendar, Clock, AlertCircle, CheckSquare } from "lucide-react";
import { format, differenceInDays, isPast } from "date-fns";
import { Link } from "react-router-dom";
import { createPageUrl } from "@/utils";

const priorityColors = {
  low: "bg-green-100 text-green-800",
  medium: "bg-yellow-100 text-yellow-800",
  high: "bg-orange-100 text-orange-800",
  urgent: "bg-red-100 text-red-800"
};

export default function MyTasks() {
  const [user, setUser] = useState(null);

  useEffect(() => {
    base44.auth.me().then(setUser).catch(() => base44.auth.redirectToLogin());
  }, []);

  const { data: allTasks = [] } = useQuery({
    queryKey: ['all-tasks'],
    queryFn: () => base44.entities.Task.list('-created_date'),
    initialData: [],
  });

  const { data: projects = [] } = useQuery({
    queryKey: ['projects'],
    queryFn: () => base44.entities.Project.list(),
    initialData: [],
  });

  const myTasks = allTasks.filter(task => task.assigned_to === user?.email);

  const categorizedTasks = {
    overdue: myTasks.filter(t => t.due_date && isPast(new Date(t.due_date)) && t.status !== 'done'),
    today: myTasks.filter(t => t.due_date && differenceInDays(new Date(t.due_date), new Date()) === 0 && t.status !== 'done'),
    upcoming: myTasks.filter(t => t.due_date && differenceInDays(new Date(t.due_date), new Date()) > 0 && t.status !== 'done'),
    noDueDate: myTasks.filter(t => !t.due_date && t.status !== 'done'),
    completed: myTasks.filter(t => t.status === 'done')
  };

  const getProjectName = (projectId) => {
    const project = projects.find(p => p.id === projectId);
    return project?.name || 'Unknown Project';
  };

  const renderTaskCard = (task) => (
    <Card key={task.id} className="hover:shadow-md transition-shadow">
      <CardContent className="p-4">
        <div className="flex items-start justify-between mb-2">
          <div className="flex-1">
            <h4 className="font-semibold mb-1">{task.title}</h4>
            <Link 
              to={`${createPageUrl("ProjectDetail")}?id=${task.project_id}`}
              className="text-sm text-indigo-600 hover:underline"
            >
              {getProjectName(task.project_id)}
            </Link>
          </div>
          <Badge className={priorityColors[task.priority]}>
            {task.priority}
          </Badge>
        </div>

        {task.description && (
          <p className="text-sm text-gray-600 mb-3 line-clamp-2">{task.description}</p>
        )}

        <div className="flex items-center gap-4 text-sm text-gray-600">
          <div className="flex items-center gap-1">
            <Badge variant="outline">{task.status.replace('_', ' ')}</Badge>
          </div>
          {task.due_date && (
            <div className="flex items-center gap-1">
              <Calendar className="w-3 h-3" />
              <span>{format(new Date(task.due_date), 'MMM d')}</span>
            </div>
          )}
          {task.estimated_hours && (
            <div className="flex items-center gap-1">
              <Clock className="w-3 h-3" />
              <span>{task.estimated_hours}h</span>
            </div>
          )}
        </div>
      </CardContent>
    </Card>
  );

  if (!user) return null;

  return (
    <div className="p-6 space-y-6">
      <div>
        <h1 className="text-3xl font-bold text-gray-900">My Tasks</h1>
        <p className="text-gray-500 mt-1">Tasks assigned to you across all projects</p>
      </div>

      <div className="grid gap-6">
        {categorizedTasks.overdue.length > 0 && (
          <Card className="border-red-200">
            <CardHeader className="bg-red-50">
              <CardTitle className="flex items-center gap-2 text-red-800">
                <AlertCircle className="w-5 h-5" />
                Overdue ({categorizedTasks.overdue.length})
              </CardTitle>
            </CardHeader>
            <CardContent className="p-4 grid gap-3">
              {categorizedTasks.overdue.map(renderTaskCard)}
            </CardContent>
          </Card>
        )}

        {categorizedTasks.today.length > 0 && (
          <Card className="border-orange-200">
            <CardHeader className="bg-orange-50">
              <CardTitle className="flex items-center gap-2 text-orange-800">
                <Calendar className="w-5 h-5" />
                Due Today ({categorizedTasks.today.length})
              </CardTitle>
            </CardHeader>
            <CardContent className="p-4 grid gap-3">
              {categorizedTasks.today.map(renderTaskCard)}
            </CardContent>
          </Card>
        )}

        {categorizedTasks.upcoming.length > 0 && (
          <Card>
            <CardHeader>
              <CardTitle>Upcoming ({categorizedTasks.upcoming.length})</CardTitle>
            </CardHeader>
            <CardContent className="p-4 grid gap-3">
              {categorizedTasks.upcoming.map(renderTaskCard)}
            </CardContent>
          </Card>
        )}

        {categorizedTasks.noDueDate.length > 0 && (
          <Card>
            <CardHeader>
              <CardTitle>No Due Date ({categorizedTasks.noDueDate.length})</CardTitle>
            </CardHeader>
            <CardContent className="p-4 grid gap-3">
              {categorizedTasks.noDueDate.map(renderTaskCard)}
            </CardContent>
          </Card>
        )}

        {categorizedTasks.completed.length > 0 && (
          <Card>
            <CardHeader className="bg-green-50">
              <CardTitle className="text-green-800">
                Completed ({categorizedTasks.completed.length})
              </CardTitle>
            </CardHeader>
            <CardContent className="p-4 grid gap-3">
              {categorizedTasks.completed.map(renderTaskCard)}
            </CardContent>
          </Card>
        )}
      </div>

      {myTasks.length === 0 && (
        <Card>
          <CardContent className="py-12 text-center">
            <CheckSquare className="w-16 h-16 mx-auto mb-4 text-gray-300" />
            <h3 className="text-xl font-semibold mb-2">No tasks assigned</h3>
            <p className="text-gray-600">You don't have any tasks assigned to you yet.</p>
          </CardContent>
        </Card>
      )}
    </div>
  );
}
