using System;
using System.Net;
using System.Net.Sockets;
using System.Text;

namespace RpsServer
{
    class Program
    {
        static void Main()
        {
            const int port = 5000;
            var listener = new TcpListener(IPAddress.Any, port);
            listener.Start();
            Console.WriteLine($"Сервер запущен на порту {port}. Ожидаем 2 клиентов...");

            // 1) Ждём двух подключений
            var client1 = listener.AcceptTcpClient();
            Console.WriteLine("Клиент 1 подключился.");
            var client2 = listener.AcceptTcpClient();
            Console.WriteLine("Клиент 2 подключился.");

            using var stream1 = client1.GetStream();
            using var stream2 = client2.GetStream();

            byte[] buf1 = new byte[256];
            byte[] buf2 = new byte[256];

            while (true)
            {
                // 2) Получаем ходы от обоих
                int len1 = stream1.Read(buf1, 0, buf1.Length);
                string move1 = Encoding.UTF8.GetString(buf1, 0, len1).Trim();
                Console.WriteLine($"Клиент1 сыграл: {move1}");

                int len2 = stream2.Read(buf2, 0, buf2.Length);
                string move2 = Encoding.UTF8.GetString(buf2, 0, len2).Trim();
                Console.WriteLine($"Клиент2 сыграл: {move2}");

                // 3) Вычисляем результат
                string result1 = GetResult(move1, move2);
                string result2 = GetResult(move2, move1);

                // 4) Отправляем каждому ответ: «Противник сыграл X, вы Y»
                Send(stream1, $"Противник сыграл {move2}, вы {result1}");
                Send(stream2, $"Противник сыграл {move1}, вы {result2}");

                Console.WriteLine("Результаты отправлены. Ждём следующий раунд...");
            }
        }

        static void Send(NetworkStream stream, string msg)
        {
            var data = Encoding.UTF8.GetBytes(msg);
            stream.Write(data, 0, data.Length);
        }

        static string GetResult(string me, string opp)
        {
            if (me == opp) return "ничья";
            if ((me == "камень" && opp == "ножницы") ||
                (me == "ножницы" && opp == "бумага") ||
                (me == "бумага" && opp == "камень"))
                return "победили";
            else
                return "проиграли";
        }
    }
}
