# E-Commerce Microservices Platform Walkthrough

## 🏆 Project Accomplishments

We have successfully built a full-stack, cloud-native E-Commerce platform using a **Microservices Architecture**. 

Here is what was accomplished:

1. **Back-end Microservices (Node.js/Express/MongoDB)**
   Created 15 independent, scalable backend services:
   - `user-service`, `authentication-service`, `product-service`, `search-service`
   - `inventory-service`, `cart-service`, `wishlist-service`, `order-service`
   - `payment-service`, `invoice-service`, `shipping-service`, `review-rating-service`
   - `recommendation-service`, `admin-service`, and an `api-gateway`.
   
   *Each service features its own [Dockerfile](file:///c:/Users/91901/OneDrive/Desktop/E-Commerce/ecommerce-microservices/frontend-ui/Dockerfile), independent `MongoDB` database setup, [package.json](file:///c:/Users/91901/OneDrive/Desktop/E-Commerce/ecommerce-microservices/frontend-ui/package.json), and API routes.*

2. **Frontend Applications (React + TailwindCSS)**
   Built a stunning, premium frontend UI (`frontend-ui`) utilizing modern glassmorphism design, vibrant gradients, and fluid micro-animations.
   - **Key Pages**: Home, Products Listing, Product Details, Cart, Wishlist, Order History, Admin Dashboard, and a Login Modal.
   - **State**: Handled via custom React Contexts (`AuthContext`, `CartContext`) interacting dynamically with the `api-gateway`.

3. **DevOps & Containerization (Docker + Kubernetes)**
   - Created a comprehensive [docker-compose.yml](file:///c:/Users/91901/OneDrive/Desktop/E-Commerce/ecommerce-microservices/docker-compose.yml) to orchestrate 16 containers (15 Node instances + 1 MongoDB replica).
   - Designed complete Kubernetes YAML manifests ([deployments.yaml](file:///c:/Users/91901/OneDrive/Desktop/E-Commerce/ecommerce-microservices/kubernetes/deployments/all-deployments.yaml), [services.yaml](file:///c:/Users/91901/OneDrive/Desktop/E-Commerce/ecommerce-microservices/kubernetes/services/all-services.yaml), [shopverse-config.yaml](file:///c:/Users/91901/OneDrive/Desktop/E-Commerce/ecommerce-microservices/kubernetes/configmaps/shopverse-config.yaml)).
   - Configured **ClusterIP** for secure internal microservice communication and **NodePort** to expose the frontend safely to the public.

## 🚀 How to Run It

### Option A: Via Docker Compose
```bash
cd ecommerce-microservices
docker-compose up --build
```
- Frontend: `http://localhost:80`
- API Gateway: `http://localhost:3000`

### Option B: Via Kubernetes (Minikube)
```bash
cd ecommerce-microservices
kubectl apply -f kubernetes/configmaps/shopverse-config.yaml
kubectl apply -f kubernetes/deployments/all-deployments.yaml
kubectl apply -f kubernetes/services/all-services.yaml
minikube service frontend-ui
```

## 🔒 Default Credentials
- **Admin Email:** `admin@example.com`
- **Password:** `123`

## 🎨 Visual Identity
The platform was styled using a custom `TailwindCSS` theme loaded with:
- Dark mode primary `bg-[#0f0f1a]` with bright pink/purple typography gradients.
- Glassmorphism effect (`backdrop-blur-xl`, `bg-white/5`).
- Fluid `animate-slide-up`, `animate-fade-in`, and scalable animations. 
