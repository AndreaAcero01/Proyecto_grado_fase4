# Proyecto_grado_fase4
aplicacion comercializacion productos agricolas
from flask import Flask, render_template, request, redirect, session, url_for

app = Flask(__name__)
app.secret_key = 'clave_secreta'

users = {}  # usuario: {password, tipo}
products = []  # lista de productos
carts = {}  # usuario: [productos]

@app.route('/')
def home():
    return render_template('home.html')

@app.route('/register', methods=['GET', 'POST'])
def register():
    if request.method == 'POST':
        user = request.form['username']
        password = request.form['password']
        role = request.form['role']
        users[user] = {'password': password, 'role': role}
        return redirect('/login')
    return render_template('register.html')

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        user = request.form['username']
        password = request.form['password']
        if user in users and users[user]['password'] == password:
            session['user'] = user
            session['role'] = users[user]['role']
            return redirect('/dashboard')
    return render_template('login.html')

@app.route('/dashboard')
def dashboard():
    if 'user' not in session:
        return redirect('/login')
    if session['role'] == 'vendedor':
        return redirect('/vender')
    else:
        return redirect('/comprar')

@app.route('/vender', methods=['GET', 'POST'])
def vender():
    if request.method == 'POST':
        producto = {
            'nombre': request.form['nombre'],
            'precio': float(request.form['precio']),
            'cantidad': int(request.form['cantidad']),
            'vendedor': session['user']
        }
        products.append(producto)
        return redirect('/vender')
    return render_template('seller.html', productos=[p for p in products if p['vendedor'] == session['user']])

@app.route('/comprar', methods=['GET', 'POST'])
def comprar():
    query = request.args.get('q', '')
    filtrados = [p for p in products if query.lower() in p['nombre'].lower()]
    return render_template('buyer.html', productos=filtrados, query=query)

@app.route('/agregar_carrito/<int:pid>')
def agregar_carrito(pid):
    user = session['user']
    if user not in carts:
        carts[user] = []
    carts[user].append(products[pid])
    return redirect('/comprar')

@app.route('/carrito')
def carrito():
    user = session['user']
    return render_template('cart.html', carrito=carts.get(user, []))

@app.route('/confirmar_pago', methods=['POST'])
def confirmar_pago():
    user = session['user']
    metodo = request.form['metodo']
    total = sum(p['precio'] for p in carts.get(user, []))
    carts[user] = []
    return render_template('confirm.html', metodo=metodo, total=total)

if __name__ == '__main__':
    app.run(debug=True)
