# curly-barnacle
تطبيق فحص رصيد توكن باستخدام Streamlit
import streamlit as st
from web3 import Web3

st.set_page_config(page_title="فحص رصيد التوكن", layout="centered")

st.title("💰 فحص رصيد توكن ERC20")

rpc_url = st.text_input(
    "رابط الشبكة (Infura / Alchemy)",
    "https://sepolia.infura.io/v3/PUT_YOUR_API_KEY_HERE"
)

token_address = st.text_input(
    "عنوان عقد التوكن",
    "0x0000000000000000000000000000000000000000"
)

wallet_address = st.text_input(
    "عنوان المحفظة",
    "0x0000000000000000000000000000000000000000"
)

decimals = st.number_input("عدد المنازل العشرية للتوكن", value=6)

abi = [
    {
        "constant": True,
        "inputs": [{"name": "_owner", "type": "address"}],
        "name": "balanceOf",
        "outputs": [{"name": "balance", "type": "uint256"}],
        "type": "function",
    }
]

if st.button("🔍 فحص الرصيد"):
    try:
        w3 = Web3(Web3.HTTPProvider(rpc_url))

        if not w3.is_connected():
            st.error("❌ فشل الاتصال بالشبكة")
        else:
            contract = w3.eth.contract(
                address=Web3.to_checksum_address(token_address),
                abi=abi
            )

            balance = contract.functions.balanceOf(
                Web3.to_checksum_address(wallet_address)
            ).call()

            human_balance = balance / (10 ** decimals)

            st.success(f"✅ الرصيد: {human_balance}")

    except Exception as e:
        st.error(f"حدث خطأ: {e}")
