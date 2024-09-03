# API-creation
from flask import Flask, jsonify, request
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)

# Configure 
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql+pymysql://root:Ramya%40123@localhost:3307/indian_banks'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)

class Bank(db.Model):
    __tablename__ = 'banks'
    id = db.Column(db.BigInteger, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    branches = db.relationship('Branch', backref='bank', lazy=True)

class Branch(db.Model):
    __tablename__ = 'branches'
    ifsc = db.Column(db.String(11), primary_key=True)  # Updated to 'ifsc'
    bank_id = db.Column(db.BigInteger, db.ForeignKey('banks.id'), nullable=False)
    branch = db.Column(db.String(100))
    address = db.Column(db.String(255))
    city = db.Column(db.String(100))
    district = db.Column(db.String(100))
    state = db.Column(db.String(100))
    bank_name = db.Column(db.String(100))

#  to get the list of all banks
@app.route('/banks', methods=['GET'])
def get_banks():
    banks = Bank.query.all()
    output = []
    for bank in banks:
        bank_data = {'id': bank.id, 'name': bank.name}
        output.append(bank_data)
    return jsonify({'banks': output})

 # to get branch details for a specific branch
@app.route('/branch/<string:ifsc>', methods=['GET'])
def get_branch_details(ifsc):
    branch = Branch.query.filter_by(ifsc=ifsc).first()
    if branch is None:
        return jsonify({'message': 'Branch not found'}), 404
    branch_data = {
        'ifsc': branch.ifsc,
        'bank_id': branch.bank_id,
        'branch': branch.branch,
        'address': branch.address,
        'city': branch.city,
        'district': branch.district,
        'state': branch.state,
        'bank_name': branch.bank_name
    }
    return jsonify({'branch': branch_data})
@app.route('/branches/ramya/state/<string:state>', methods=['GET'])
def get_branches_by_state(state):
    branches = Branch.query.filter_by(state=state).all()
    if not branches:
        return jsonify({'message': 'No branches found in this state'}), 404
    output = []
    for branch1 in branches:
        branch_data1 = {
            'ifsc': branch1.ifsc,
            'bank_id': branch1.bank_id,
            'branch': branch1.branch,
            'address': branch1.address,
            'city': branch1.city,
            'district': branch1.district,
            'state': branch1.state,
            'bank_name': branch1.bank_name
        }
        output.append(branch_data1)
    return jsonify({'branches': output})
   
if __name__ == '__main__':
    print("Running")
    app.run(debug=True)
