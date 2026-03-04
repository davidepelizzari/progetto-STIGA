# progetto-STIGA


Email: davide.pelizzari@barsantigalilei.edu.it
password: Stiga08?

using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;
using System.Net.Http;
using Newtonsoft.Json;
using System.Net.Http.Headers;
using Newtonsoft.Json.Linq;

namespace WindowsFormsApp1
{
    public partial class Form1 : Form
    {
        private string Token;
        private string Uuid;

        public Form1()
        {
            InitializeComponent();
        }

        private void Form1_Load(object sender, EventArgs e)
        {

        }

        private void labelEmail_Click(object sender, EventArgs e)
        {

        }

        private void Email_TextChanged(object sender, EventArgs e)
        {

        }

        private void labelPassword_Click(object sender, EventArgs e)
        {

        }

        private void Password_TextChanged(object sender, EventArgs e)
        {

        }

        private async void button1_Click(object sender, EventArgs e)
        {
            string mail = Email.Text;
            string pass = Password.Text;
            string token = await RetrieveIdToken(mail, pass);
            string uuid = await GetRobot(token);

            Token = token;
            Uuid = uuid;


            if (token != null)
            {
                labelEmail.Visible = false;
                labelPassword.Visible = false;
                Email.Visible = false;
                Password.Visible = false;
                button1.Visible = false;
                Nome.Visible = true;
                avviaRobot.Visible = true;
                fermaRobot.Visible = true;
                statoRobot.Visible = true;
                NomeRobot.Visible = true;
            }


        }



        private static readonly HttpClient client = new HttpClient();

        private async Task<string> RetrieveIdToken(string Mail, string Pass)
        {
            try
            {
                string url = "https://www.googleapis.com/identitytoolkit/v3/relyingparty/verifyPassword?key=AIzaSyCPtRBU_hwWZYsguHp9ucGrfNac0kXR6ug";

                var requestBody = new
                {
                    email = Mail,
                    password = Pass,
                    returnSecureToken = true
                };

                string jsonBody = JsonConvert.SerializeObject(requestBody);

                var content = new StringContent(jsonBody, Encoding.UTF8, "application/json");

                HttpResponseMessage response = await client.PostAsync(url, content);

                string responseJson = await response.Content.ReadAsStringAsync();

                if (response.IsSuccessStatusCode)
                {
                    FirebaseAuthResponse authResponse = JsonConvert.DeserializeObject<FirebaseAuthResponse>(responseJson);
                    Nome.Text = Nome.Text + $" {authResponse.displayName}";
                    return authResponse.idToken;
                }
                else
                {
                    MessageBox.Show("Errore autenticazione: " + responseJson);
                    return null;
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show("Errore di rete: " + ex.Message);
                return null;
            }
        }

        public class FirebaseAuthResponse
        {
            public string idToken { get; set; }
            public string email { get; set; }
            public string refreshToken { get; set; }
            public string expiresIn { get; set; }
            public string localId { get; set; }
            public string displayName {  get; set; }
        }

        private async Task<string> GetRobot(string Token)
        {
            string url = "https://connectivity-production.stiga.com/api/garage/integration";

            using (HttpClient client = new HttpClient())
            {
                try
                {
                    client.DefaultRequestHeaders.Authorization =
                        new AuthenticationHeaderValue("Bearer", Token);

                    HttpResponseMessage response = await client.GetAsync(url);

                    if (response.IsSuccessStatusCode)
                    {
                        string json = await response.Content.ReadAsStringAsync();

                        Root result = JsonConvert.DeserializeObject<Root>(json);

                        if (result?.data != null && result.data.Count > 0)
                        {
                            var device = result.data[0];
                            NomeRobot.Text = $"{device.attributes.name}";
                            return device.attributes.uuid;
                        }

                        return null;
                    }
                    else
                    {
                        MessageBox.Show($"Errore HTTP: {response.StatusCode}");
                        return null;
                    }
                }
                catch (Exception ex)
                {
                    MessageBox.Show("Errore: " + ex.Message);
                    return null;
                }
            }
        }

        public class Root
        {
            public List<DeviceData> data { get; set; }
        }

        public class DeviceData
        {
            public string type { get; set; }
            public DeviceAttributes attributes { get; set; }
        }

        public class DeviceAttributes
        {
            public string uuid { get; set; }
            public string product_code { get; set; }
            public string serial_number { get; set; }
            public string name { get; set; }
            public string device_type { get; set; }
        }

        private void Name_Click(object sender, EventArgs e)
        {

        }

        private void NomeRobot_Click(object sender, EventArgs e)
        {

        }

        private async void avviaRobot_Click(object sender, EventArgs e)
        {
            string testo = await AvviaRobot();
        }

        private async Task<string> AvviaRobot()
        {
            using (HttpClient client = new HttpClient())
            {
                string url = $"https://connectivity-production.stiga.com/api/devices/{Uuid}/command/startsession";

                try
                {
                    client.DefaultRequestHeaders.Authorization =
                        new AuthenticationHeaderValue("Bearer", Token);

                    HttpResponseMessage response = await client.GetAsync(url);

                    if (response.IsSuccessStatusCode)
                    {
                        statoRobot.Text = "Stato: in movimento";
                        return "";
                    }
                    else
                    {
                        return "";
                    }
                }
                catch (Exception ex)
                {
                    Console.WriteLine("Errore: " + ex.Message);
                    return "";
                }
            }
        }


        private async void fermaRobot_Click(object sender, EventArgs e)
        {
            string stringa = await FermaRobot();
        }

        private async Task<string> FermaRobot()
        {
            using (HttpClient client = new HttpClient())
            {
                string url = $"https://connectivity-production.stiga.com/api/devices/{Uuid}/command/endsession";

                try
                {
                    client.DefaultRequestHeaders.Authorization =
                        new AuthenticationHeaderValue("Bearer", Token);

                    HttpResponseMessage response = await client.GetAsync(url);

                    if (response != null)
                    {
                        return "";
                    }
                    else
                    {
                        statoRobot.Text = "Stato: in attesa";
                        return "";
                    }
                }
                catch (Exception ex)
                {
                    Console.WriteLine("Errore: " + ex.Message);
                    return "";
                }
            }
        }

        private void statoRobot_Click(object sender, EventArgs e)
        {

        }
    }
}

