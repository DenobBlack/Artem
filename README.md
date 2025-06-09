using System;
using System.Net;
using System.Net.Sockets;
using System.Text;

namespace RpsClient
{
    class Program
    {
        static void Main()
        {
            const string host = "localhost";
            const int port = 5000;
            var client = new TcpClient();
            client.Connect(host, port);
            Console.WriteLine("Подключились к серверу.");

            using var stream = client.GetStream();
            while (true)
            {
                Console.Write("Ваш ход (камень/ножницы/бумага): ");
                string move = Console.ReadLine()?.Trim().ToLower();
                if (move != "камень" && move != "ножницы" && move != "бумага")
                {
                    Console.WriteLine("Неверный ход, попробуйте снова.");
                    continue;
                }

                // 1) Отправляем ход
                var data = Encoding.UTF8.GetBytes(move);
                stream.Write(data, 0, data.Length);

                // 2) Ждём и читаем ответ
                byte[] buf = new byte[256];
                int count = stream.Read(buf, 0, buf.Length);
                string response = Encoding.UTF8.GetString(buf, 0, count);
                Console.WriteLine($"Сервер: {response}");
                Console.WriteLine();
            }
        }
    }
}
