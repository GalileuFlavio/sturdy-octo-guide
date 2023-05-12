# sturdy-octo-guide
Automatizar o agendamento eletrônico de Grus em um site governamental
import time
import requests
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.action_chains import ActionChains

# URL do site de login do gov.br
login_url = 'https://sso.acesso.gov.br/login?client_id=d3c457bb0bd04f60b2abf8de61ab7aa1&authorization_redirect_uri=https%3A%2F%2Fsistemas.dpc.mar.mil.br%2Fsisap%2Fagendamento%2F%23%2F&scope=openid+email+phone+profile&response_type=code&state=b6df8d6a02a44e1aa7ec6fcf35b80a07&nonce=caec9b1d3a4c41d9ac2ec482e95584c1'

# Dados de login
username = 'seu_username'
password = 'sua_senha'

# Configura o driver do Selenium
driver = webdriver.Chrome()
wait = WebDriverWait(driver, 10)

# Faz login no site do gov.br
driver.get(login_url)
wait.until(EC.presence_of_element_located((By.ID, 'username')))
driver.find_element_by_id('username').send_keys(username)
driver.find_element_by_id('password').send_keys(password)
driver.find_element_by_id('btn-login').click()

# Espera até que o login seja concluído e o H-CAPTCHA apareça
wait.until(EC.url_contains('https://sistemas.dpc.mar.mil.br/sisap/agendamento/#/'))
wait.until(EC.presence_of_element_located((By.ID, 'hcaptcha_widget')))

# Resolve o H-CAPTCHA
hcaptcha_widget = driver.find_element_by_id('hcaptcha_widget')
actions = ActionChains(driver)
actions.move_to_element(hcaptcha_widget)
actions.click(hcaptcha_widget)
actions.perform()
time.sleep(5)  # Aguarda a verificação do H-CAPTCHA

# Envia uma solicitação HTTP POST com os dados de agendamento
url = 'https://sistemas.dpc.mar.mil.br/sisap/agendamento/api/servicos/agendamento'
data = {
    # Insira os dados do agendamento aqui
}
headers = {
    'Content-Type': 'application/json',
    'Authorization': f'Bearer {driver.execute_script("return sessionStorage.getItem('access_token')")}'
}
response = requests.post(url, json=data, headers=headers)

# Fecha o driver do Selenium
driver.quit()
